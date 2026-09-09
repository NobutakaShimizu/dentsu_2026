<script setup lang="ts">
import { computed, onUnmounted, ref, watch } from 'vue'
import { useNav, useSlideContext } from '@slidev/client'
import MathTex from './Math.vue'

const N = 8
const W = 12
const correctSet = new Set([
  '0,2', '0,9', '1,3', '1,8', '2,4',
  '3,6', '3,11', '4,0', '4,10', '5,1',
  '6,2', '6,7', '7,3', '7,8', '8,9',
  '9,4', '9,11', '10,5', '11,0', '11,7',
])

const noiseCells = Array.from({ length: W * W }, (_, t) => ({
  i: Math.floor(t / W),
  j: t % W,
})).filter(p => !correctSet.has(`${p.i},${p.j}`))

const NOISE_COUNT = noiseCells.length

const { $clicks } = useSlideContext()
const { isPrintMode } = useNav()

function makeMat(rows: number, cols: number) {
  return Array.from({ length: rows }, () => Array.from({ length: cols }, () => 0))
}

const A = makeMat(N, N)
const B = makeMat(N, N)
const AB = makeMat(N, N)
const C1 = makeMat(N, N)
const C2 = makeMat(N, N)
const Ap = makeMat(W, N)
const Bp = makeMat(N, W)
const exact = makeMat(W, W)

const extraA = Array.from({ length: (W - N) * N }, (_, t) => ({
  i: N + Math.floor(t / N),
  j: t % N,
}))
const extraB = Array.from({ length: N * (W - N) }, (_, t) => ({
  i: t % N,
  j: N + Math.floor(t / N),
}))
const extraMax = extraA.length + extraB.length

const stage = computed(() => {
  if (isPrintMode.value)
    return 3
  return $clicks.value ?? 0
})

const showLift = ref(false)
const showApBp = ref(false)
const extraCount = ref(0)
const showMArrow = ref(false)
const showProducts = ref(false)
const noiseShown = ref(0)
const showDec = ref(false)
const showList = ref(false)

let token = 0

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
  showLift.value = false
  showApBp.value = false
  extraCount.value = 0
  showMArrow.value = false
  showProducts.value = false
  noiseShown.value = 0
  showDec.value = false
  showList.value = false
}

function applyFinal() {
  token += 1
  showLift.value = true
  showApBp.value = true
  extraCount.value = extraMax
  showMArrow.value = true
  showProducts.value = true
  noiseShown.value = NOISE_COUNT
  showDec.value = true
  showList.value = true
}

async function playLift(t: number) {
  showLift.value = true
  showApBp.value = true
  extraCount.value = 0
  for (let k = 1; k <= extraMax; k++) {
    if (t !== token)
      return
    extraCount.value = k
    await delay(16, t)
  }
}

async function playProducts(t: number) {
  showMArrow.value = true
  await delay(220, t)
  if (t !== token)
    return
  showProducts.value = true
  noiseShown.value = 0
  await delay(320, t)
  for (let k = 1; k <= NOISE_COUNT; k++) {
    if (t !== token)
      return
    noiseShown.value = k
    await delay(12, t)
  }
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
    if (n === 1) {
      token += 1
      showMArrow.value = false
      showProducts.value = false
      noiseShown.value = 0
      showDec.value = false
      showList.value = false
      if (!showApBp.value || extraCount.value < extraMax)
        playLift(++token)
      else
        extraCount.value = extraMax
      return
    }
    showLift.value = true
    showApBp.value = true
    extraCount.value = extraMax
    if (n === 2) {
      token += 1
      showDec.value = false
      showList.value = false
      if (!showProducts.value || noiseShown.value < NOISE_COUNT)
        playProducts(++token)
      else
        noiseShown.value = NOISE_COUNT
      return
    }
    showMArrow.value = true
    showProducts.value = true
    noiseShown.value = NOISE_COUNT
    showDec.value = true
    showList.value = true
  },
  { immediate: true },
)

onUnmounted(() => {
  token += 1
})

function extraAVisible(i: number, j: number) {
  if (i < N)
    return true
  const idx = extraA.findIndex(p => p.i === i && p.j === j)
  return idx >= 0 && extraCount.value >= idx + 1
}

function extraBVisible(i: number, j: number) {
  if (j < N)
    return true
  const idx = extraB.findIndex(p => p.i === i && p.j === j)
  return idx >= 0 && extraCount.value >= extraA.length + idx + 1
}

function isNoise(i: number, j: number) {
  const idx = noiseCells.findIndex(p => p.i === i && p.j === j)
  return idx >= 0 && noiseShown.value >= idx + 1
}
</script>

<template>
  <div class="ewr">
    <div v-click class="ewr-click" aria-hidden="true" />
    <div v-click class="ewr-click" aria-hidden="true" />
    <div v-click class="ewr-click" aria-hidden="true" />

    <div class="ewr-left-top">
      <div class="ewr-mat is-in is-sm">
        <div class="ewr-name"><MathTex tex="A" /></div>
        <div class="ewr-grid">
          <div v-for="(row, r) in A" :key="`A-${r}`" class="ewr-row">
            <div v-for="(_val, c) in row" :key="`A-${r}-${c}`" class="ewr-cell" />
          </div>
        </div>
      </div>
      <div class="ewr-mat is-in is-sm">
        <div class="ewr-name"><MathTex tex="B" /></div>
        <div class="ewr-grid">
          <div v-for="(row, r) in B" :key="`B-${r}`" class="ewr-row">
            <div v-for="(_val, c) in row" :key="`B-${r}-${c}`" class="ewr-cell" />
          </div>
        </div>
      </div>
    </div>

    <div class="ewr-harrow" :class="{ 'is-in': showLift }">
      <span class="ewr-harrow-label">lifting</span>
      <span class="ewr-harrow-shaft"><span class="ewr-harrow-line" /></span>
    </div>

    <div class="ewr-right-top">
      <div class="ewr-mat is-tall" :class="{ 'is-in': showApBp }">
        <div class="ewr-name"><MathTex tex="A'" /></div>
        <div class="ewr-grid is-coded">
          <div v-for="(row, r) in Ap" :key="`Ap-${r}`" class="ewr-row">
            <div
              v-for="(val, c) in row"
              :key="`Ap-${r}-${c}`"
              class="ewr-cell"
              :class="{
                'is-msg': r < N,
                'is-extra': extraAVisible(r, c) && r >= N,
                'is-wait': !extraAVisible(r, c),
              }"
            />
          </div>
        </div>
      </div>
      <div class="ewr-mat is-wide" :class="{ 'is-in': showApBp }">
        <div class="ewr-name"><MathTex tex="B'" /></div>
        <div class="ewr-grid is-coded">
          <div v-for="(row, r) in Bp" :key="`Bp-${r}`" class="ewr-row">
            <div
              v-for="(val, c) in row"
              :key="`Bp-${r}-${c}`"
              class="ewr-cell"
              :class="{
                'is-msg': c < N,
                'is-extra': extraBVisible(r, c) && c >= N,
                'is-wait': !extraBVisible(r, c),
              }"
            />
          </div>
        </div>
      </div>
    </div>

    <div class="ewr-varrow" :class="{ 'is-in': showMArrow }">
      <span class="ewr-varrow-label">Algorithm <MathTex tex="M" /></span>
      <span class="ewr-varrow-line" />
    </div>

    <div class="ewr-left-bot">
      <p class="ewr-note" :class="{ 'is-in': showList }">list of matrices</p>
      <div class="ewr-list" :class="{ 'is-in': showList }">
        <div class="ewr-mat is-sm is-ghost is-in">
          <div class="ewr-grid">
            <div v-for="(row, r) in C1" :key="`C1-${r}`" class="ewr-row">
              <div v-for="(_val, c) in row" :key="`C1-${r}-${c}`" class="ewr-cell" />
            </div>
          </div>
        </div>
        <div class="ewr-mat is-sm is-in">
          <div class="ewr-name"><MathTex tex="AB" /></div>
          <div class="ewr-grid is-clean">
            <div v-for="(row, r) in AB" :key="`AB-${r}`" class="ewr-row">
              <div v-for="(_val, c) in row" :key="`AB-${r}-${c}`" class="ewr-cell is-clean" />
            </div>
          </div>
        </div>
        <div class="ewr-mat is-sm is-ghost is-in">
          <div class="ewr-grid">
            <div v-for="(row, r) in C2" :key="`C2-${r}`" class="ewr-row">
              <div v-for="(_val, c) in row" :key="`C2-${r}-${c}`" class="ewr-cell" />
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="ewr-harrow ewr-harrow-rev" :class="{ 'is-in': showDec }">
      <span class="ewr-harrow-label">approximate<br>list-decoding</span>
      <span class="ewr-harrow-shaft"><span class="ewr-harrow-line" /></span>
    </div>

    <div class="ewr-m-out">
      <div class="ewr-mat is-lg" :class="{ 'is-in': showProducts }">
        <div class="ewr-name"><MathTex tex="M(A',B')" /></div>
        <div class="ewr-grid">
          <div v-for="(row, r) in exact" :key="`M-${r}`" class="ewr-row">
            <div
              v-for="(_val, c) in row"
              :key="`M-${r}-${c}`"
              class="ewr-cell"
              :class="{ 'is-noise': isNoise(r, c) }"
            />
          </div>
        </div>
      </div>

      <div class="ewr-true-side" :class="{ 'is-in': showProducts }">
        <div class="ewr-approx" :class="{ 'is-in': showProducts, 'is-noisy': noiseShown > 0 }">
          <MathTex tex="\approx" />
        </div>

        <div class="ewr-name ewr-enc-name"><MathTex tex="\mathrm{Enc}(AB)" /></div>
        <div class="ewr-grid ewr-enc-grid">
          <div v-for="(row, r) in exact" :key="`P-${r}`" class="ewr-row">
            <div v-for="(_val, c) in row" :key="`P-${r}-${c}`" class="ewr-cell" />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.ewr {
  position: relative;
  display: grid;
  grid-template-columns: max-content 8.6rem max-content;
  grid-template-rows: auto 3.1rem auto;
  align-items: center;
  justify-content: center;
  column-gap: 0.4rem;
  row-gap: 0.2rem;
  margin: 0.4rem auto 0;
  padding-right: 10.2rem;
  width: fit-content;
  max-width: 100%;
}

.ewr-click {
  position: absolute;
  width: 0;
  height: 0;
  overflow: hidden;
}

.ewr-left-top,
.ewr-right-top {
  display: flex;
  align-items: flex-end;
  justify-content: center;
  gap: 0.4rem;
}

.ewr-left-top { grid-column: 1; grid-row: 1; }
.ewr-harrow { grid-column: 2; grid-row: 1; }
.ewr-right-top { grid-column: 3; grid-row: 1; }
.ewr-varrow { grid-column: 3; grid-row: 2; justify-self: center; }
.ewr-left-bot { grid-column: 1; grid-row: 3; }
.ewr-harrow-rev { grid-column: 2; grid-row: 3; }
.ewr-m-out {
  grid-column: 3;
  grid-row: 3;
  justify-self: center;
  position: relative;
}

.ewr-m-out > .ewr-mat {
  display: grid;
  grid-template-rows: 1.15rem auto;
  justify-items: center;
  row-gap: 0.1rem;
}

.ewr-true-side {
  position: absolute;
  left: calc(100% + 0.4rem);
  top: 0;
  display: grid;
  grid-template-columns: auto auto;
  grid-template-rows: 1.15rem auto;
  column-gap: 0.4rem;
  row-gap: 0.1rem;
  align-items: center;
  justify-items: center;
  opacity: 0;
  pointer-events: none;
}

.ewr-true-side.is-in {
  opacity: 1;
  pointer-events: auto;
  transition: opacity 0.28s ease;
}

.ewr-approx {
  grid-column: 1;
  grid-row: 2;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.05rem;
  font-size: 1.1rem;
  color: #546e7a;
  line-height: 1;
}

.ewr-approx.is-in {
  opacity: 1;
}

.ewr-approx.is-noisy {
  color: #c2185b;
  font-weight: 700;
}

.ewr-enc-name {
  grid-column: 2;
  grid-row: 1;
}

.ewr-enc-grid {
  grid-column: 2;
  grid-row: 2;
}

.ewr-mat {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.1rem;
  opacity: 0;
  transform: translateY(8px);
}

.ewr-mat.is-in {
  opacity: 1;
  transform: none;
  transition: opacity 0.28s ease, transform 0.28s ease;
}

.ewr-mat.is-ghost {
  opacity: 0.38;
}

.ewr-list.is-in .ewr-mat.is-ghost {
  opacity: 0.38;
}

.ewr-name {
  font-weight: 700;
  font-size: 0.88rem;
  color: #37474f;
  line-height: 1.15rem;
  min-height: 1.15rem;
  display: flex;
  align-items: center;
  justify-content: center;
}

.ewr-grid {
  display: flex;
  flex-direction: column;
  gap: 1px;
  background: #b0bec5;
  border: 1px solid #78909c;
  padding: 1px;
}

.ewr-grid.is-coded {
  box-shadow: inset 0 0 0 1px rgba(21, 101, 192, 0.12);
}

.ewr-grid.is-clean {
  border-color: #90a4ae;
}

.ewr-row {
  display: flex;
  gap: 1px;
}

.ewr-cell {
  display: flex;
  align-items: center;
  justify-content: center;
  background: #fff;
  color: #455a64;
  font-family: 'Fira Code', monospace;
  line-height: 1;
}

.ewr-mat.is-sm .ewr-cell {
  width: 6px;
  height: 6px;
  font-size: 0;
}

.ewr-mat.is-tall .ewr-cell,
.ewr-mat.is-wide .ewr-cell,
.ewr-mat.is-lg .ewr-cell,
.ewr-enc-grid .ewr-cell {
  width: 5px;
  height: 5px;
  font-size: 0;
}

.ewr-cell.is-wait {
  background: #eceff1;
  color: transparent;
}

.ewr-cell.is-noise {
  background: #f8bbd0;
  color: #880e4f;
  font-weight: 700;
}

.ewr-harrow {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 100%;
  min-width: 8.6rem;
  opacity: 0;
  transform: translateX(-8px);
}

.ewr-harrow.is-in {
  opacity: 1;
  transform: none;
  transition: opacity 0.24s ease, transform 0.24s ease;
}

.ewr-harrow-label {
  font-family: 'Roboto', sans-serif;
  font-size: 0.68rem;
  font-weight: 600;
  color: #546e7a;
  line-height: 1.15;
  text-align: center;
  white-space: nowrap;
  margin-bottom: 0.1rem;
}

.ewr-harrow-rev .ewr-harrow-label {
  white-space: normal;
}

.ewr-harrow-shaft {
  position: relative;
  display: block;
  width: 100%;
  height: 0.7rem;
}

.ewr-harrow-line {
  position: absolute;
  left: 0.1rem;
  right: 0.1rem;
  top: 50%;
  border-top: 2px solid #546e7a;
}

.ewr-harrow-line::after {
  content: '';
  position: absolute;
  right: -1px;
  top: 50%;
  border-top: 5px solid transparent;
  border-bottom: 5px solid transparent;
  border-left: 9px solid #546e7a;
  transform: translateY(-50%);
}

.ewr-harrow-rev .ewr-harrow-line::after {
  right: auto;
  left: -1px;
  border-left: none;
  border-right: 9px solid #546e7a;
}

.ewr-varrow {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.28rem;
  height: 100%;
  opacity: 0;
}

.ewr-varrow.is-in {
  opacity: 1;
  transition: opacity 0.24s ease;
}

.ewr-varrow-label {
  display: flex;
  align-items: center;
  gap: 0.2rem;
  font-family: 'Roboto', sans-serif;
  font-weight: 700;
  font-size: 0.78rem;
  color: #37474f;
  white-space: nowrap;
}

.ewr-varrow-line {
  position: relative;
  display: block;
  width: 0;
  height: 2.15rem;
  border-left: 2px solid #546e7a;
}

.ewr-varrow-line::after {
  content: '';
  position: absolute;
  left: -6px;
  bottom: -2px;
  border-left: 5px solid transparent;
  border-right: 5px solid transparent;
  border-top: 9px solid #546e7a;
}

.ewr-list {
  display: flex;
  align-items: flex-end;
  gap: 0.28rem;
  opacity: 0;
}

.ewr-list.is-in {
  opacity: 1;
  transition: opacity 0.24s ease;
}

.ewr-note {
  margin: 0;
  font-family: 'Roboto', sans-serif;
  font-size: 0.62rem;
  font-weight: 600;
  color: #546e7a;
  text-align: center;
  line-height: 1.2;
  opacity: 0;
}

.ewr-note.is-in {
  opacity: 1;
  transition: opacity 0.2s ease;
}
</style>
