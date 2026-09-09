<script setup lang="ts">
import { computed, onUnmounted, ref, watch } from 'vue'
import { useNav, useSlideContext } from '@slidev/client'
import MathTex from './Math.vue'

const N = 4
const N2 = 6
const Q = 5
const NOISE_COUNT = 28

const { $clicks } = useSlideContext()
const { isPrintMode } = useNav()

const props = withDefaults(defineProps<{
  topOnly?: boolean
}>(), {
  topOnly: false,
})

function mulberry32(seed: number) {
  let s = seed >>> 0
  return () => {
    s = (s + 0x6D2B79F5) >>> 0
    let t = Math.imul(s ^ (s >>> 15), 1 | s)
    t = (t + Math.imul(t ^ (t >>> 7), 61 | t)) ^ t
    return ((t ^ (t >>> 14)) >>> 0) / 4294967296
  }
}

function makeMat(rows: number, cols: number, seed: number) {
  const rng = mulberry32(seed)
  return Array.from({ length: rows }, () =>
    Array.from({ length: cols }, () => Math.floor(rng() * Q)),
  )
}

function mul(a: number[][], b: number[][]) {
  const m = a.length
  const k = a[0]!.length
  const n = b[0]!.length
  return Array.from({ length: m }, (_, i) =>
    Array.from({ length: n }, (_, j) => {
      let s = 0
      for (let t = 0; t < k; t++)
        s += a[i]![t]! * b[t]![j]!
      return s % Q
    }),
  )
}

function makeL(seed: number) {
  const rng = mulberry32(seed)
  return Array.from({ length: N2 }, (_, i) =>
    Array.from({ length: N }, (_, j) => {
      if (i < N)
        return i === j ? 1 : 0
      return Math.floor(rng() * Q)
    }),
  )
}

function transpose(m: number[][]) {
  return Array.from({ length: m[0]!.length }, (_, j) =>
    Array.from({ length: m.length }, (_, i) => m[i]![j]!),
  )
}

const extraLA = Array.from({ length: (N2 - N) * N }, (_, t) => ({
  i: N + Math.floor(t / N),
  j: t % N,
}))
const extraBLt = Array.from({ length: N * (N2 - N) }, (_, t) => ({
  i: Math.floor(t / (N2 - N)),
  j: N + (t % (N2 - N)),
}))

const A = makeMat(N, N, 21)
const B = makeMat(N, N, 34)
const L = makeL(81)
const LA = mul(L, A)
const BLt = mul(B, transpose(L))
const exact = mul(LA, BLt)
const AB = mul(A, B)

const rngNoise = mulberry32(55)
const noiseOrder = Array.from({ length: N2 * N2 }, (_, t) => t)
for (let t = noiseOrder.length - 1; t > 0; t--) {
  const u = Math.floor(rngNoise() * (t + 1))
  const tmp = noiseOrder[t]!
  noiseOrder[t] = noiseOrder[u]!
  noiseOrder[u] = tmp
}
const noiseCells = noiseOrder.slice(0, NOISE_COUNT).map(t => ({
  i: Math.floor(t / N2),
  j: t % N2,
}))
const noiseSet = new Set(noiseCells.map(p => p.i * N2 + p.j))

const M = exact.map((row, i) =>
  row.map((v, j) => {
    if (!noiseSet.has(i * N2 + j))
      return v
    let w = v
    while (w === v)
      w = Math.floor(rngNoise() * Q)
    return w
  }),
)

const clicks = computed(() => $clicks.value ?? 0)
const stage = computed(() => {
  if (props.topOnly)
    return 1
  if (isPrintMode.value)
    return 3
  return clicks.value
})

const showEnc = ref(false)
const showCoded = ref(false)
const extraCount = ref(0)
const showMArrow = ref(false)
const showProducts = ref(false)
const noiseShown = ref(0)
const showDec = ref(false)
const showAB = ref(false)

let token = 0
const extraMax = Math.max(extraLA.length, extraBLt.length)

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
  showEnc.value = false
  showCoded.value = false
  extraCount.value = 0
  showMArrow.value = false
  showProducts.value = false
  noiseShown.value = 0
  showDec.value = false
  showAB.value = false
}

function applyFinal() {
  token += 1
  showEnc.value = true
  showCoded.value = true
  extraCount.value = extraMax
  showMArrow.value = true
  showProducts.value = true
  noiseShown.value = NOISE_COUNT
  showDec.value = true
  showAB.value = true
}

async function playEncode(t: number) {
  showEnc.value = true
  showCoded.value = true
  extraCount.value = 0
  for (let k = 1; k <= extraMax; k++) {
    if (t !== token)
      return
    extraCount.value = k
    await delay(28, t)
  }
}

async function playProducts(t: number) {
  showMArrow.value = true
  await delay(220, t)
  if (t !== token)
    return
  showProducts.value = true
  noiseShown.value = 0
  await delay(380, t)
  for (let k = 1; k <= NOISE_COUNT; k++) {
    if (t !== token)
      return
    noiseShown.value = k
    await delay(22, t)
  }
}

watch(
  stage,
  (n) => {
    if (props.topOnly) {
      token += 1
      showEnc.value = true
      showCoded.value = true
      extraCount.value = extraMax
      showMArrow.value = false
      showProducts.value = false
      noiseShown.value = 0
      showDec.value = false
      showAB.value = false
      return
    }
    if (isPrintMode.value) {
      applyFinal()
      return
    }
    if (n <= 0) {
      reset()
      return
    }

    if (n === 1) {
      token += 1
      showMArrow.value = false
      showProducts.value = false
      noiseShown.value = 0
      showDec.value = false
      showAB.value = false
      if (!showCoded.value || extraCount.value < extraMax)
        playEncode(++token)
      else
        extraCount.value = extraMax
      return
    }

    showEnc.value = true
    showCoded.value = true
    extraCount.value = extraMax

    if (n === 2) {
      token += 1
      showDec.value = false
      showAB.value = false
      if (!showProducts.value || noiseShown.value < NOISE_COUNT)
        playProducts(++token)
      else
        noiseShown.value = NOISE_COUNT
      return
    }

    showMArrow.value = true
    showProducts.value = true
    if (noiseShown.value < NOISE_COUNT)
      noiseShown.value = NOISE_COUNT
    showDec.value = true
    showAB.value = true
  },
  { immediate: true },
)

onUnmounted(() => {
  token += 1
})

function extraVisibleLA(i: number, j: number) {
  if (i < N)
    return true
  const idx = extraLA.findIndex(p => p.i === i && p.j === j)
  return idx >= 0 && extraCount.value >= idx + 1
}

function extraVisibleBLt(i: number, j: number) {
  if (j < N)
    return true
  const idx = extraBLt.findIndex(p => p.i === i && p.j === j)
  return idx >= 0 && extraCount.value >= idx + 1
}

function isNoise(i: number, j: number) {
  const idx = noiseCells.findIndex(p => p.i === i && p.j === j)
  return idx >= 0 && noiseShown.value >= idx + 1
}

function mVal(i: number, j: number) {
  return isNoise(i, j) ? M[i]![j]! : exact[i]![j]!
}
</script>

<template>
  <div class="er" :class="{ 'is-top': topOnly }">
    <template v-if="!topOnly">
      <div v-click class="er-click" aria-hidden="true" />
      <div v-click class="er-click" aria-hidden="true" />
      <div v-click class="er-click" aria-hidden="true" />
    </template>

    <div class="er-left-top">
      <div class="er-mat is-in is-sm">
        <div class="er-name"><MathTex tex="A" /></div>
        <div class="er-grid">
          <div v-for="(row, r) in A" :key="`A-${r}`" class="er-row">
            <div v-for="(val, c) in row" :key="`A-${r}-${c}`" class="er-cell">{{ val }}</div>
          </div>
        </div>
      </div>
      <div class="er-mat is-in is-sm">
        <div class="er-name"><MathTex tex="B" /></div>
        <div class="er-grid">
          <div v-for="(row, r) in B" :key="`B-${r}`" class="er-row">
            <div v-for="(val, c) in row" :key="`B-${r}-${c}`" class="er-cell">{{ val }}</div>
          </div>
        </div>
      </div>
    </div>

    <div class="er-harrow" :class="{ 'is-in': showEnc }">
      <span class="er-harrow-label">encoding</span>
      <span class="er-harrow-shaft">
        <span class="er-harrow-line" />
      </span>
    </div>

    <div class="er-right-top">
      <div class="er-mat is-lg" :class="{ 'is-in': showCoded }">
        <div class="er-name"><MathTex tex="LA" /></div>
        <div class="er-grid is-coded">
          <div v-for="(row, r) in LA" :key="`LA-${r}`" class="er-row">
            <div
              v-for="(val, c) in row"
              :key="`LA-${r}-${c}`"
              class="er-cell"
              :class="{
                'is-msg': r < N,
                'is-extra': extraVisibleLA(r, c) && r >= N,
                'is-wait': !extraVisibleLA(r, c),
              }"
            >{{ extraVisibleLA(r, c) ? val : '' }}</div>
          </div>
        </div>
      </div>
      <div class="er-mat is-lg" :class="{ 'is-in': showCoded }">
        <div class="er-name"><MathTex tex="BL^\top" /></div>
        <div class="er-grid is-coded">
          <div v-for="(row, r) in BLt" :key="`BLt-${r}`" class="er-row">
            <div
              v-for="(val, c) in row"
              :key="`BLt-${r}-${c}`"
              class="er-cell"
              :class="{
                'is-msg': c < N,
                'is-extra': extraVisibleBLt(r, c) && c >= N,
                'is-wait': !extraVisibleBLt(r, c),
              }"
            >{{ extraVisibleBLt(r, c) ? val : '' }}</div>
          </div>
        </div>
      </div>
    </div>

    <template v-if="!topOnly">
    <div class="er-varrow" :class="{ 'is-in': showMArrow }">
      <span class="er-varrow-label"><MathTex tex="M" /></span>
      <span class="er-varrow-line" />
    </div>

    <div class="er-left-bot">
      <div class="er-mat is-sm" :class="{ 'is-in': showAB }">
        <div class="er-name"><MathTex tex="AB" /></div>
        <div class="er-grid is-clean">
          <div v-for="(row, r) in AB" :key="`AB-${r}`" class="er-row">
            <div v-for="(val, c) in row" :key="`AB-${r}-${c}`" class="er-cell is-clean">{{ val }}</div>
          </div>
        </div>
        <p class="er-sub" :class="{ 'is-in': showAB }">を含むリスト</p>
      </div>
    </div>

    <div class="er-harrow er-harrow-rev" :class="{ 'is-in': showDec }">
      <span class="er-harrow-label">list-decoding</span>
      <span class="er-harrow-shaft">
        <span class="er-harrow-line" />
      </span>
    </div>

    <div class="er-m-out">
      <div class="er-mat is-lg" :class="{ 'is-in': showProducts }">
        <div class="er-name"><MathTex tex="M(LA,BL^\top)" /></div>
        <div class="er-grid">
          <div v-for="(row, r) in exact" :key="`M-${r}`" class="er-row">
            <div
              v-for="(_val, c) in row"
              :key="`M-${r}-${c}`"
              class="er-cell"
              :class="{ 'is-noise': isNoise(r, c) }"
            >{{ mVal(r, c) }}</div>
          </div>
        </div>
      </div>

      <div class="er-true-side">
        <div class="er-approx" :class="{ 'is-in': showProducts, 'is-noisy': noiseShown > 0 }">
          <MathTex tex="\approx" />
        </div>

        <div class="er-mat is-lg" :class="{ 'is-in': showProducts }">
          <div class="er-name"><MathTex tex="L\,AB\,L^\top" /></div>
          <div class="er-grid">
            <div v-for="(row, r) in exact" :key="`P-${r}`" class="er-row">
              <div v-for="(val, c) in row" :key="`P-${r}-${c}`" class="er-cell">{{ val }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
    </template>
  </div>
</template>

<style scoped>
.er {
  position: relative;
  display: grid;
  grid-template-columns: max-content 10.6rem max-content;
  grid-template-rows: auto 3.6rem auto;
  align-items: center;
  justify-content: center;
  column-gap: 0.5rem;
  row-gap: 0.3rem;
  margin: 0.35rem auto 0;
  padding-right: 9.6rem;
  width: fit-content;
  max-width: 100%;
}

.er.is-top {
  grid-template-rows: auto;
  padding-right: 0;
  margin-top: 0.55rem;
}

.er-click {
  position: absolute;
  width: 0;
  height: 0;
  overflow: hidden;
}

.er-left-top,
.er-right-top,
.er-left-bot {
  display: flex;
  align-items: flex-end;
  justify-content: center;
  gap: 0.45rem;
}

.er-left-top {
  grid-column: 1;
  grid-row: 1;
}

.er-harrow {
  grid-column: 2;
  grid-row: 1;
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 0;
  width: 100%;
  min-width: 10.6rem;
  opacity: 0;
  transform: translateX(-8px);
}

.er-right-top {
  grid-column: 3;
  grid-row: 1;
}

.er-varrow {
  grid-column: 3;
  grid-row: 2;
  justify-self: center;
}

.er-left-bot {
  grid-column: 1;
  grid-row: 3;
}

.er-harrow-rev {
  grid-column: 2;
  grid-row: 3;
}

.er-m-out {
  grid-column: 3;
  grid-row: 3;
  justify-self: center;
  position: relative;
}

.er-true-side {
  position: absolute;
  left: calc(100% + 0.4rem);
  bottom: 0;
  display: flex;
  align-items: flex-end;
  gap: 0.4rem;
}

.er-mat {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.12rem;
  opacity: 0;
  transform: translateY(8px);
  pointer-events: none;
}

.er-mat.is-in {
  opacity: 1;
  transform: none;
  pointer-events: auto;
  transition: opacity 0.28s ease, transform 0.28s ease;
}

.er-name {
  font-family: 'Roboto', sans-serif;
  font-weight: 700;
  font-size: 1.12rem;
  color: #37474f;
  line-height: 1.1;
  min-height: 1.25rem;
}

.er-name :deep(.math) {
  font-size: 0.95em;
}

.er-sub {
  margin: 0.12rem 0 0;
  font-size: 0.88rem;
  font-weight: 600;
  color: #546e7a;
  opacity: 0;
}

.er-sub.is-in {
  opacity: 1;
  transition: opacity 0.24s ease;
}

.er-grid {
  display: flex;
  flex-direction: column;
  gap: 1px;
  background: #b0bec5;
  border: 1px solid #78909c;
  padding: 1px;
}

.er-grid.is-coded {
  box-shadow: inset 0 0 0 1px rgba(21, 101, 192, 0.12);
}

.er-grid.is-clean {
  border-color: #90a4ae;
}

.er-row {
  display: flex;
  gap: 1px;
}

.er-cell {
  display: flex;
  align-items: center;
  justify-content: center;
  background: #fff;
  color: #455a64;
  font-family: 'Fira Code', monospace;
  line-height: 1;
  transition:
    background-color 0.18s ease,
    color 0.18s ease,
    opacity 0.16s ease;
}

.er-mat.is-sm .er-cell {
  width: 22px;
  height: 22px;
  font-size: 11px;
}

.er-mat.is-lg .er-cell {
  width: 19px;
  height: 19px;
  font-size: 10px;
}

.er-cell.is-msg,
.er-cell.is-extra {
  background: #fff;
}

.er-cell.is-wait {
  background: #eceff1;
  color: transparent;
}

.er-cell.is-noise {
  background: #f8bbd0;
  color: #880e4f;
  font-weight: 700;
}

.er-cell.is-clean {
  background: #fff;
  color: #37474f;
}

.er-harrow.is-in {
  opacity: 1;
  transform: none;
  transition: opacity 0.24s ease, transform 0.24s ease;
}

.er-harrow-label {
  font-family: 'Roboto', sans-serif;
  font-size: 0.86rem;
  font-weight: 600;
  letter-spacing: 0.02em;
  color: #546e7a;
  line-height: 1;
  white-space: nowrap;
  margin-bottom: 0.12rem;
}

.er-harrow-shaft {
  position: relative;
  display: block;
  width: 100%;
  height: 0.7rem;
}

.er-harrow-line {
  position: absolute;
  left: 0.1rem;
  right: 0.1rem;
  top: 50%;
  border-top: 2px solid #546e7a;
}

.er-harrow-line::after {
  content: '';
  position: absolute;
  right: -1px;
  top: 50%;
  width: 0;
  height: 0;
  border-top: 5px solid transparent;
  border-bottom: 5px solid transparent;
  border-left: 9px solid #546e7a;
  transform: translateY(-50%);
}

.er-harrow-rev .er-harrow-line::after {
  right: auto;
  left: -1px;
  border-left: none;
  border-right: 9px solid #546e7a;
}

.er-varrow {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.28rem;
  height: 100%;
  opacity: 0;
  transform: translateY(-6px);
}

.er-varrow.is-in {
  opacity: 1;
  transform: none;
  transition: opacity 0.24s ease, transform 0.24s ease;
}

.er-varrow-label {
  font-family: 'Roboto', sans-serif;
  font-weight: 700;
  font-size: 1.15rem;
  color: #37474f;
  line-height: 1;
}

.er-varrow-line {
  position: relative;
  display: block;
  width: 0;
  height: 2.8rem;
  border-left: 2px solid #546e7a;
}

.er-varrow-line::after {
  content: '';
  position: absolute;
  left: -6px;
  bottom: -2px;
  width: 0;
  height: 0;
  border-left: 5px solid transparent;
  border-right: 5px solid transparent;
  border-top: 9px solid #546e7a;
}

.er-approx {
  align-self: center;
  padding: 0 0.15rem 0.15rem;
  font-size: 1.4rem;
  color: #546e7a;
  opacity: 0;
  transform: scale(0.85);
}

.er-approx.is-in {
  opacity: 1;
  transform: none;
  transition: opacity 0.22s ease, transform 0.22s ease, color 0.2s ease;
}

.er-approx.is-noisy {
  color: #c2185b;
  font-weight: 700;
}
</style>
