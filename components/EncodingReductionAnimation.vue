<script setup lang="ts">
import { computed, onUnmounted, ref, watch } from 'vue'
import { useNav, useSlideContext } from '@slidev/client'
import { QrCodeDataType, encode as encodeQr } from 'uqr'
import MathTex from './Math.vue'

const N = 4
const Q = 5

const qrOpts = {
  ecc: 'Q' as const,
  border: 0,
  minVersion: 1,
  maxVersion: 2,
}

const qrA = encodeQr('Enc(A)', qrOpts)
const qrB = encodeQr('Enc(B)', qrOpts)
const qrP = encodeQr('Enc(AB)', qrOpts)
const QR = qrA.size

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

function makeMat(n: number, seed: number) {
  const rng = mulberry32(seed)
  return Array.from({ length: n }, () =>
    Array.from({ length: n }, () => Math.floor(rng() * Q)),
  )
}

function mul(a: number[][], b: number[][]) {
  const n = a.length
  return Array.from({ length: n }, (_, i) =>
    Array.from({ length: n }, (_, j) => {
      let s = 0
      for (let k = 0; k < n; k++)
        s += a[i]![k]! * b[k]![j]!
      return s % Q
    }),
  )
}

const encodeCells = Array.from({ length: QR * QR }, (_, t) => ({
  i: Math.floor(t / QR),
  j: t % QR,
}))

function shuffleCells<T>(arr: T[], seed: number) {
  const rng = mulberry32(seed)
  const out = arr.slice()
  for (let t = out.length - 1; t > 0; t--) {
    const u = Math.floor(rng() * (t + 1))
    const tmp = out[t]!
    out[t] = out[u]!
    out[u] = tmp
  }
  return out
}

const extraOrderA = shuffleCells(encodeCells, 101)
const extraOrderB = shuffleCells(encodeCells, 202)
const extraIndexA = new Map(extraOrderA.map((p, k) => [`${p.i},${p.j}`, k]))
const extraIndexB = new Map(extraOrderB.map((p, k) => [`${p.i},${p.j}`, k]))

const A = makeMat(N, 21)
const B = makeMat(N, 34)
const AB = mul(A, B)

const noiseCandidates = encodeCells.filter(p => qrP.types[p.i]![p.j] === QrCodeDataType.Data)
const NOISE_COUNT = Math.max(18, Math.round(noiseCandidates.length * 0.16))
const noiseCells = shuffleCells(noiseCandidates, 55).slice(0, NOISE_COUNT)

const clicks = computed(() => $clicks.value ?? 0)
const stage = computed(() => {
  if (isPrintMode.value)
    return 3
  return clicks.value
})

const showEnc = ref(false)
const showApBp = ref(false)
const extraCount = ref(0)
const encodeTick = ref(0)
const encoding = ref(false)
const showMArrow = ref(false)
const showProducts = ref(false)
const noiseShown = ref(0)
const showDec = ref(false)
const showAB = ref(false)

let token = 0
let encodeTickId = 0

function delay(ms: number, t: number) {
  return new Promise<void>((resolve) => {
    window.setTimeout(() => {
      if (t === token)
        resolve()
    }, ms)
  })
}

function clearEncodeTick() {
  if (encodeTickId) {
    window.clearInterval(encodeTickId)
    encodeTickId = 0
  }
}

function stopEncodeTick() {
  clearEncodeTick()
  encoding.value = false
}

function reset() {
  token += 1
  stopEncodeTick()
  showEnc.value = false
  showApBp.value = false
  extraCount.value = 0
  encodeTick.value = 0
  showMArrow.value = false
  showProducts.value = false
  noiseShown.value = 0
  showDec.value = false
  showAB.value = false
}

function applyFinal() {
  token += 1
  stopEncodeTick()
  showEnc.value = true
  showApBp.value = true
  extraCount.value = encodeCells.length
  encodeTick.value = 0
  showMArrow.value = true
  showProducts.value = true
  noiseShown.value = NOISE_COUNT
  showDec.value = true
  showAB.value = true
}

async function playEncode(t: number) {
  showEnc.value = true
  showApBp.value = true
  extraCount.value = 0
  encodeTick.value = 0
  clearEncodeTick()
  encoding.value = true

  const scrambleMs = 320
  const lockMs = 720
  const started = performance.now()

  await new Promise<void>((resolve) => {
    encodeTickId = window.setInterval(() => {
      if (t !== token) {
        window.clearInterval(encodeTickId)
        encodeTickId = 0
        resolve()
        return
      }
      encodeTick.value += 1
      const elapsed = performance.now() - started
      const locked = elapsed < scrambleMs
        ? 0
        : Math.min(
            encodeCells.length,
            Math.round(((elapsed - scrambleMs) / lockMs) * encodeCells.length),
          )
      extraCount.value = locked
      if (locked >= encodeCells.length) {
        window.clearInterval(encodeTickId)
        encodeTickId = 0
        resolve()
      }
    }, 18)
  })

  if (t === token)
    stopEncodeTick()
}

async function playProducts(t: number) {
  showMArrow.value = true
  await delay(220, t)
  if (t !== token)
    return
  showProducts.value = true
  noiseShown.value = 0
  await delay(420, t)
  for (let k = 1; k <= NOISE_COUNT; k++) {
    if (t !== token)
      return
    noiseShown.value = k
    await delay(18, t)
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
      showAB.value = false
      if (!showApBp.value || extraCount.value < encodeCells.length)
        playEncode(++token)
      else
        extraCount.value = encodeCells.length
      return
    }

    stopEncodeTick()
    showEnc.value = true
    showApBp.value = true
    extraCount.value = encodeCells.length

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
  stopEncodeTick()
})

function extraIndex(which: 'A' | 'B', i: number, j: number) {
  const map = which === 'A' ? extraIndexA : extraIndexB
  return map.get(`${i},${j}`) ?? -1
}

function extraLocked(which: 'A' | 'B', i: number, j: number) {
  const idx = extraIndex(which, i, j)
  return idx >= 0 && extraCount.value >= idx + 1
}

function extraComputing(which: 'A' | 'B', i: number, j: number) {
  return showApBp.value && encoding.value && !extraLocked(which, i, j)
}

function qrCellClass(which: 'A' | 'B', i: number, j: number, data: boolean[][]) {
  if (extraComputing(which, i, j))
    return 'is-computing'
  if (!extraLocked(which, i, j))
    return 'is-wait'
  return data[i]![j]! ? 'is-qr-black' : 'is-qr-white'
}

function noiseIndex(i: number, j: number) {
  return noiseCells.findIndex(p => p.i === i && p.j === j)
}

function isNoise(i: number, j: number) {
  const idx = noiseIndex(i, j)
  return idx >= 0 && noiseShown.value >= idx + 1
}

function mCellClass(i: number, j: number) {
  if (isNoise(i, j))
    return 'is-noise'
  return qrP.data[i]![j]! ? 'is-qr-black' : 'is-qr-white'
}

function pCellClass(i: number, j: number) {
  return qrP.data[i]![j]! ? 'is-qr-black' : 'is-qr-white'
}
</script>

<template>
  <div class="er">
    <div v-click class="er-click" aria-hidden="true" />
    <div v-click class="er-click" aria-hidden="true" />
    <div v-click class="er-click" aria-hidden="true" />

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
      <div class="er-mat is-qr" :class="{ 'is-in': showApBp }">
        <div class="er-name"><MathTex tex="A'" /></div>
        <div class="er-grid is-qr">
          <div v-for="(row, r) in qrA.data" :key="`Ap-${r}`" class="er-row">
            <div
              v-for="(_bit, c) in row"
              :key="`Ap-${r}-${c}`"
              class="er-cell"
              :class="qrCellClass('A', r, c, qrA.data)"
            />
          </div>
        </div>
      </div>
      <div class="er-mat is-qr" :class="{ 'is-in': showApBp }">
        <div class="er-name"><MathTex tex="B'" /></div>
        <div class="er-grid is-qr">
          <div v-for="(row, r) in qrB.data" :key="`Bp-${r}`" class="er-row">
            <div
              v-for="(_bit, c) in row"
              :key="`Bp-${r}-${c}`"
              class="er-cell"
              :class="qrCellClass('B', r, c, qrB.data)"
            />
          </div>
        </div>
      </div>
    </div>

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
      </div>
    </div>

    <div class="er-harrow er-harrow-rev" :class="{ 'is-in': showDec }">
      <span class="er-harrow-label">decoding</span>
      <span class="er-harrow-shaft">
        <span class="er-harrow-line" />
      </span>
    </div>

    <div class="er-m-out">
      <div class="er-mat is-qr" :class="{ 'is-in': showProducts }">
        <div class="er-name"><MathTex tex="M(A',B')" /></div>
        <div class="er-grid is-qr">
          <div v-for="(row, r) in qrP.data" :key="`M-${r}`" class="er-row">
            <div
              v-for="(_bit, c) in row"
              :key="`M-${r}-${c}`"
              class="er-cell"
              :class="mCellClass(r, c)"
            />
          </div>
        </div>
      </div>

      <div class="er-true-side">
        <div class="er-approx" :class="{ 'is-in': showProducts, 'is-noisy': noiseShown > 0 }">
          <MathTex tex="\approx" />
        </div>

        <div class="er-mat is-qr" :class="{ 'is-in': showProducts }">
          <div class="er-name"><MathTex tex="A'\cdot B'" /></div>
          <div class="er-grid is-qr">
            <div v-for="(row, r) in qrP.data" :key="`P-${r}`" class="er-row">
              <div
                v-for="(_bit, c) in row"
                :key="`P-${r}-${c}`"
                class="er-cell"
                :class="pCellClass(r, c)"
              />
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.er {
  position: relative;
  display: grid;
  grid-template-columns: max-content 8.8rem max-content;
  grid-template-rows: auto 3rem auto;
  align-items: center;
  justify-content: center;
  column-gap: 0.35rem;
  row-gap: 0.2rem;
  margin: 0.45rem auto 0;
  padding-right: 8.4rem;
  width: fit-content;
  max-width: 100%;
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
  gap: 0.55rem;
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
  min-width: 8.8rem;
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
  font-size: 0.92rem;
  color: #37474f;
  line-height: 1.1;
  min-height: 1.1rem;
}

.er-name :deep(.math) {
  font-size: 0.95em;
}

.er-grid {
  display: flex;
  flex-direction: column;
  gap: 1px;
  background: #b0bec5;
  border: 1px solid #78909c;
  padding: 1px;
}

.er-grid.is-qr {
  gap: 0;
  padding: 3px;
  background: #fff;
  border: 1px solid #90a4ae;
}

.er-grid.is-clean {
  border-color: #90a4ae;
}

.er-row {
  display: flex;
  gap: 1px;
}

.er-grid.is-qr .er-row {
  gap: 0;
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
    background-color 0.16s ease,
    color 0.16s ease;
}

.er-mat.is-sm .er-cell {
  width: 15px;
  height: 15px;
  font-size: 8px;
}

.er-mat.is-qr .er-cell {
  width: 4px;
  height: 4px;
}

.er-cell.is-wait {
  background: #eceff1;
}

.er-cell.is-computing {
  background: #ffcc80;
}

.er-cell.is-qr-black {
  background: #212121;
}

.er-cell.is-qr-white {
  background: #fff;
}

.er-cell.is-noise {
  background: #f8bbd0;
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
  font-size: 0.7rem;
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
  font-size: 0.95rem;
  color: #37474f;
  line-height: 1;
}

.er-varrow-line {
  position: relative;
  display: block;
  width: 0;
  height: 2.35rem;
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
  font-size: 1.15rem;
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
