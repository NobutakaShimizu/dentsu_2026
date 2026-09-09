<script setup lang="ts">
import { computed, onUnmounted, ref, watch } from 'vue'
import { useNav, useSlideContext } from '@slidev/client'

const N = 8
const Q = 5
const TOTAL = N * N
const HITS = 26

const { $clicks } = useSlideContext()
const { isPrintMode } = useNav()

function mulberry32(seed: number) {
  let s = seed >>> 0
  return () => {
    s = (s + 0x6D2B79F5) >>> 0
    let t = Math.imul(s ^ (s >>> 15), 1 | s)
    t = (t + Math.imul(t ^ (t >>> 7), 61 | t)) ^ t
    return ((t ^ (t >>> 14)) >>> 0) / 4294967296
  }
}

function makeMat(seed: number) {
  const rng = mulberry32(seed)
  return Array.from({ length: N }, () =>
    Array.from({ length: N }, () => Math.floor(rng() * Q)),
  )
}

function mul(a: number[][], b: number[][]) {
  return Array.from({ length: N }, (_, i) =>
    Array.from({ length: N }, (_, j) => {
      let s = 0
      for (let k = 0; k < N; k++)
        s += a[i]![k]! * b[k]![j]!
      return s % Q
    }),
  )
}

function makeApprox(exact: number[][], seed: number) {
  const rng = mulberry32(seed)
  const idxs = Array.from({ length: TOTAL }, (_, t) => t)
  for (let t = idxs.length - 1; t > 0; t--) {
    const u = Math.floor(rng() * (t + 1))
    const tmp = idxs[t]!
    idxs[t] = idxs[u]!
    idxs[u] = tmp
  }
  const keep = new Set(idxs.slice(0, HITS))
  const c = exact.map(row => [...row])
  const match: boolean[][] = Array.from({ length: N }, () => Array.from({ length: N }, () => false))
  for (let t = 0; t < TOTAL; t++) {
    const i = Math.floor(t / N)
    const j = t % N
    if (keep.has(t)) {
      match[i]![j] = true
      continue
    }
    let v = exact[i]![j]!
    while (v === exact[i]![j])
      v = Math.floor(rng() * Q)
    c[i]![j] = v
  }
  return { c, match }
}

const A = makeMat(21)
const B = makeMat(34)
const exact = mul(A, B)
const { c: C, match } = makeApprox(exact, 55)
const hitCells = Array.from({ length: TOTAL }, (_, t) => ({
  i: Math.floor(t / N),
  j: t % N,
})).filter(p => match[p.i]![p.j])

const showAB = ref(false)
const showArrow = ref(false)
const showC = ref(false)
const revealedHits = ref(0)
const showCount = ref(false)

let token = 0

const clicks = computed(() => $clicks.value ?? 0)
const stage = computed(() => {
  if (isPrintMode.value)
    return 3
  return clicks.value
})

function delay(ms: number, t: number) {
  return new Promise<void>((resolve) => {
    window.setTimeout(() => {
      if (t === token)
        resolve()
    }, ms)
  })
}

function reset() {
  token += 1
  showAB.value = false
  showArrow.value = false
  showC.value = false
  revealedHits.value = 0
  showCount.value = false
}

function applyFinal() {
  token += 1
  showAB.value = true
  showArrow.value = true
  showC.value = true
  revealedHits.value = HITS
  showCount.value = true
}

async function playRevealC(t: number) {
  showArrow.value = true
  await delay(280, t)
  if (t !== token)
    return
  showC.value = true
}

async function playHighlight(t: number) {
  showArrow.value = true
  showC.value = true
  for (let h = 1; h <= HITS; h++) {
    if (t !== token)
      return
    revealedHits.value = h
    await delay(28, t)
  }
  if (t !== token)
    return
  showCount.value = true
}

watch(
  stage,
  (n) => {
    if (isPrintMode.value) {
      applyFinal()
      return
    }
    if (n <= 0) {
      reset()
      return
    }
    showAB.value = true
    if (n === 1) {
      token += 1
      showArrow.value = false
      showC.value = false
      revealedHits.value = 0
      showCount.value = false
      return
    }
    if (n === 2) {
      token += 1
      revealedHits.value = 0
      showCount.value = false
      if (!showC.value)
        playRevealC(++token)
      return
    }
    if (revealedHits.value < HITS)
      playHighlight(++token)
  },
  { immediate: true },
)

onUnmounted(() => {
  token += 1
})

function isHit(i: number, j: number) {
  return match[i]![j] && revealedHits.value >= hitCells.findIndex(p => p.i === i && p.j === j) + 1
}
</script>

<template>
  <div class="amm">
    <div v-click class="amm-click-slot" aria-hidden="true" />
    <div v-click class="amm-click-slot" aria-hidden="true" />
    <div v-click class="amm-click-slot" aria-hidden="true" />

    <div class="amm-board">
      <div class="amm-mat" :class="{ 'is-in': showAB }">
        <div class="amm-name">A</div>
        <div class="amm-grid">
          <div v-for="(row, r) in A" :key="`A-${r}`" class="amm-row">
            <div v-for="(val, c) in row" :key="`A-${r}-${c}`" class="amm-cell">{{ val }}</div>
          </div>
        </div>
      </div>

      <div class="amm-mat" :class="{ 'is-in': showAB }">
        <div class="amm-name">B</div>
        <div class="amm-grid">
          <div v-for="(row, r) in B" :key="`B-${r}`" class="amm-row">
            <div v-for="(val, c) in row" :key="`B-${r}-${c}`" class="amm-cell">{{ val }}</div>
          </div>
        </div>
      </div>

      <div class="amm-arrow" :class="{ 'is-in': showArrow }">→</div>

      <div class="amm-mat" :class="{ 'is-in': showC }">
        <div class="amm-name">C</div>
        <div class="amm-grid">
          <div v-for="(row, r) in C" :key="`C-${r}`" class="amm-row">
            <div
              v-for="(val, c) in row"
              :key="`C-${r}-${c}`"
              class="amm-cell"
              :class="{ 'is-hit': isHit(r, c) }"
            >{{ val }}</div>
          </div>
        </div>
      </div>
    </div>

    <p v-if="showCount" class="amm-count">
      成功成分数 / 全成分数 = {{ HITS }} / {{ TOTAL }}
    </p>
  </div>
</template>

<style scoped>
.amm {
  position: relative;
  margin-top: 0.55rem;
}

.amm-click-slot {
  position: absolute;
  width: 0;
  height: 0;
  overflow: hidden;
}

.amm-board {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.7rem;
  min-height: 11.2rem;
}

.amm-mat {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.2rem;
  opacity: 0;
  transform: translateY(8px);
  pointer-events: none;
}

.amm-mat.is-in {
  opacity: 1;
  transform: none;
  pointer-events: auto;
  transition: opacity 0.28s ease, transform 0.28s ease;
}

.amm-name {
  font-family: 'Roboto', sans-serif;
  font-weight: 700;
  font-style: italic;
  font-size: 1.05rem;
  color: #37474f;
  line-height: 1;
}

.amm-grid {
  display: flex;
  flex-direction: column;
  gap: 1px;
  background: #b0bec5;
  border: 1px solid #78909c;
  padding: 1px;
}

.amm-row {
  display: flex;
  gap: 1px;
}

.amm-cell {
  width: 16px;
  height: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #fff;
  color: #455a64;
  font-size: 8px;
  font-family: 'Fira Code', monospace;
  line-height: 1;
  transition: background-color 0.16s ease, color 0.16s ease;
}

.amm-cell.is-hit {
  background: #bbdefb;
  color: #0d47a1;
  font-weight: 700;
}

.amm-arrow {
  font-size: 1.7rem;
  font-weight: 600;
  color: #546e7a;
  padding-top: 1.05rem;
  opacity: 0;
  transform: translateX(-8px);
}

.amm-arrow.is-in {
  opacity: 1;
  transform: none;
  transition: opacity 0.22s ease, transform 0.22s ease;
}

.amm-count {
  margin: 0.45rem 0 0;
  text-align: center;
  font-family: 'Roboto', sans-serif;
  font-size: 1rem;
  font-weight: 600;
  color: #1565c0;
}
</style>
