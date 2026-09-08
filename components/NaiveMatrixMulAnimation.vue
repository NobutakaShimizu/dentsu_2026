<script setup lang="ts">
import { computed, onUnmounted, ref, watch } from 'vue'
import { useNav, useSlideContext } from '@slidev/client'

const N = 10

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
    Array.from({ length: N }, () => Math.floor(rng() * 6)),
  )
}

function emptyC(): (number | null)[][] {
  return Array.from({ length: N }, () => Array.from({ length: N }, () => null))
}

function fullC(a: number[][], b: number[][]) {
  return Array.from({ length: N }, (_, i) =>
    Array.from({ length: N }, (_, j) => {
      let s = 0
      for (let k = 0; k < N; k++)
        s += a[i]![k]! * b[k]![j]!
      return s
    }),
  )
}

const A = makeMat(7)
const B = makeMat(13)
const Cfinal = fullC(A, B)

const C = ref<(number | null)[][]>(emptyC())
const iIdx = ref(0)
const jIdx = ref(0)
const kIdx = ref(-1)
const playing = ref(false)
const finished = ref(false)

let token = 0

const started = computed(() => isPrintMode.value || ($clicks.value ?? 0) >= 1)

function delayMs(i: number, j: number) {
  if (i === 0 && j === 0)
    return 150
  if (i === 0 && j === 1)
    return 70
  if (i === 0 && j === 2)
    return 38
  if (i === 0)
    return 16
  if (i === 1)
    return 7
  if (i === 2)
    return 4
  return 2
}

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
  playing.value = false
  finished.value = false
  iIdx.value = 0
  jIdx.value = 0
  kIdx.value = -1
  C.value = emptyC()
}

function applyFinal() {
  token += 1
  playing.value = false
  finished.value = true
  iIdx.value = N - 1
  jIdx.value = N - 1
  kIdx.value = N - 1
  C.value = Cfinal.map(row => [...row])
}

async function run() {
  const t = ++token
  playing.value = true
  finished.value = false
  C.value = emptyC()

  for (let i = 0; i < N; i++) {
    for (let j = 0; j < N; j++) {
      if (t !== token)
        return
      iIdx.value = i
      jIdx.value = j
      kIdx.value = -1
      let sum = 0
      const next = C.value.map(row => [...row])
      next[i]![j] = 0
      C.value = next

      for (let k = 0; k < N; k++) {
        if (t !== token)
          return
        kIdx.value = k
        sum += A[i]![k]! * B[k]![j]!
        const grid = C.value.map(row => [...row])
        grid[i]![j] = sum
        C.value = grid
        await delay(delayMs(i, j), t)
      }
    }
  }

  if (t !== token)
    return
  finished.value = true
  playing.value = false
}

function cellClass(kind: 'A' | 'B' | 'C', r: number, c: number) {
  const i = iIdx.value
  const j = jIdx.value
  const k = kIdx.value
  const active = started.value && playing.value && !finished.value
  const cls = ['nmm-cell']

  if (kind === 'A') {
    if (active && r === i)
      cls.push('is-row')
    if (active && r === i && c === k)
      cls.push('is-scan', 'is-scan-a')
  }
  if (kind === 'B') {
    if (active && c === j)
      cls.push('is-col')
    if (active && r === k && c === j)
      cls.push('is-scan', 'is-scan-b')
  }
  if (kind === 'C') {
    const val = C.value[r]![c]
    if (val != null)
      cls.push(r === i && c === j && !finished.value ? 'is-target' : 'is-filled')
    if (active && r === i && c === j && !finished.value)
      cls.push('is-writing')
  }
  return cls
}

watch(
  started,
  (on) => {
    if (isPrintMode.value) {
      applyFinal()
      return
    }
    if (on)
      run()
    else
      reset()
  },
  { immediate: true },
)

onUnmounted(() => {
  token += 1
})
</script>

<template>
  <div class="nmm">
    <div v-click class="nmm-click-slot" aria-hidden="true" />

    <div class="nmm-board">
      <div class="nmm-mat">
        <div class="nmm-name">A</div>
        <div class="nmm-grid" aria-label="matrix A">
          <div
            v-for="(row, r) in A"
            :key="`A-${r}`"
            class="nmm-row"
          >
            <div
              v-for="(val, c) in row"
              :key="`A-${r}-${c}`"
              :class="cellClass('A', r, c)"
            >{{ val }}</div>
          </div>
        </div>
      </div>

      <div class="nmm-op">×</div>

      <div class="nmm-mat">
        <div class="nmm-name">B</div>
        <div class="nmm-grid" aria-label="matrix B">
          <div
            v-for="(row, r) in B"
            :key="`B-${r}`"
            class="nmm-row"
          >
            <div
              v-for="(val, c) in row"
              :key="`B-${r}-${c}`"
              :class="cellClass('B', r, c)"
            >{{ val }}</div>
          </div>
        </div>
      </div>

      <div class="nmm-op">=</div>

      <div class="nmm-mat">
        <div class="nmm-name">C</div>
        <div class="nmm-grid" aria-label="matrix C">
          <div
            v-for="(row, r) in C"
            :key="`C-${r}`"
            class="nmm-row"
          >
            <div
              v-for="(val, c) in row"
              :key="`C-${r}-${c}`"
              :class="cellClass('C', r, c)"
            >{{ val == null ? '' : val }}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.nmm {
  position: relative;
  margin-top: 0.35rem;
}

.nmm-click-slot {
  position: absolute;
  width: 0;
  height: 0;
  overflow: hidden;
}

.nmm-board {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.55rem;
}

.nmm-mat {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.2rem;
}

.nmm-name {
  font-family: 'Roboto', sans-serif;
  font-weight: 700;
  font-style: italic;
  font-size: 1.05rem;
  color: #37474f;
  line-height: 1;
}

.nmm-grid {
  display: flex;
  flex-direction: column;
  gap: 1px;
  background: #b0bec5;
  border: 1px solid #78909c;
  padding: 1px;
}

.nmm-row {
  display: flex;
  gap: 1px;
}

.nmm-cell {
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
  transition: background-color 0.12s ease, color 0.12s ease, box-shadow 0.12s ease;
}

.nmm-cell.is-row {
  background: #bbdefb;
  color: #0d47a1;
}

.nmm-cell.is-col {
  background: #ffe0b2;
  color: #e65100;
}

.nmm-cell.is-scan-a {
  background: #1565c0;
  color: #fff;
  font-weight: 700;
  box-shadow: inset 0 0 0 1px #0d47a1;
}

.nmm-cell.is-scan-b {
  background: #ef6c00;
  color: #fff;
  font-weight: 700;
  box-shadow: inset 0 0 0 1px #e65100;
}

.nmm-cell.is-filled {
  background: #e8f5e9;
  color: #1b5e20;
}

.nmm-cell.is-target,
.nmm-cell.is-writing {
  background: #f8bbd0;
  color: #880e4f;
  font-weight: 700;
}

.nmm-op {
  font-size: 1.35rem;
  font-weight: 600;
  color: #546e7a;
  padding-top: 1.05rem;
}
</style>
