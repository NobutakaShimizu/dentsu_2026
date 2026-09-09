<script setup lang="ts">
import { computed, onUnmounted, ref, watch } from 'vue'
import { useNav, useSlideContext } from '@slidev/client'
import MathTex from './Math.vue'

const N = 5
const Q = 5
const X = [1, 4, 2, 0, 3]
const STEP_MS = 220

const pairs = Array.from({ length: N * N }, (_, t) => {
  const i = Math.floor(t / N)
  const j = t % N
  return { i, j, val: (X[i]! + X[j]!) % Q }
})

const { $clicks } = useSlideContext()
const { isPrintMode } = useNav()

const stage = computed(() => {
  if (isPrintMode.value)
    return 1
  return $clicks.value ?? 0
})

const shown = ref(0)
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
  shown.value = 0
}

function applyFinal() {
  token += 1
  shown.value = pairs.length
}

async function playFill(t: number) {
  shown.value = 0
  await delay(180, t)
  for (let k = 1; k <= pairs.length; k++) {
    if (t !== token)
      return
    shown.value = k
    await delay(STEP_MS, t)
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
    playFill(++token)
  },
  { immediate: true },
)

onUnmounted(() => {
  token += 1
})

const active = computed(() => {
  if (shown.value <= 0 || shown.value > pairs.length)
    return null
  return pairs[shown.value - 1]!
})

const done = computed(() => shown.value >= pairs.length)

function xActive(k: number) {
  const a = active.value
  if (!a || done.value)
    return false
  return k === a.i || k === a.j
}

function encVisible(t: number) {
  return t < shown.value
}

function encActive(t: number) {
  return !done.value && t === shown.value - 1
}
</script>

<template>
  <div class="ds">
    <div v-click class="ds-click" aria-hidden="true" />

    <div class="ds-row">
      <div class="ds-name"><MathTex tex="x" /></div>
      <div class="ds-vec">
        <div
          v-for="(v, k) in X"
          :key="`x-${k}`"
          class="ds-cell"
          :class="{ 'is-src': xActive(k) }"
        >{{ v }}</div>
      </div>
    </div>

    <div class="ds-row ds-arrow-row">
      <div class="ds-name" aria-hidden="true" />
      <div class="ds-arrow">
        <span class="ds-arrow-line" />
      </div>
    </div>

    <div class="ds-row">
      <div class="ds-name"><MathTex tex="\mathrm{Enc}(x)" /></div>
      <div class="ds-vec is-enc">
        <div
          v-for="(p, t) in pairs"
          :key="`e-${t}`"
          class="ds-cell"
          :class="{
            'is-wait': !encVisible(t),
            'is-on': encVisible(t),
            'is-now': encActive(t),
          }"
        >{{ encVisible(t) ? p.val : '' }}</div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.ds {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.18rem;
  margin: 0.55rem auto 0.1rem;
  width: fit-content;
  max-width: 100%;
}

.ds-click {
  position: absolute;
  width: 0;
  height: 0;
  overflow: hidden;
}

.ds-row {
  display: flex;
  align-items: center;
  gap: 0.55rem;
}

.ds-name {
  flex: 0 0 5.4rem;
  text-align: right;
  font-size: 0.95rem;
  font-weight: 700;
  color: #37474f;
  line-height: 1;
}

.ds-vec {
  display: flex;
  gap: 1px;
  background: #b0bec5;
  border: 1px solid #78909c;
  padding: 1px;
}

.ds-cell {
  width: 1.45rem;
  height: 1.45rem;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #fff;
  color: #455a64;
  font-family: 'Fira Code', monospace;
  font-size: 0.78rem;
  line-height: 1;
  transition: background-color 0.18s ease, color 0.18s ease, box-shadow 0.18s ease;
}

.ds-cell.is-src {
  background: #f8bbd0;
  color: #c2185b;
  font-weight: 700;
}

.ds-cell.is-wait {
  background: #eceff1;
  color: transparent;
}

.ds-cell.is-on {
  background: #fff;
  color: #37474f;
}

.ds-cell.is-now {
  background: #f8bbd0;
  color: #c2185b;
  font-weight: 700;
}

.ds-arrow-row {
  align-items: stretch;
}

.ds-arrow {
  display: flex;
  justify-content: center;
  width: calc(5 * 1.45rem + 8px);
}

.ds-arrow-line {
  position: relative;
  display: block;
  width: 0;
  height: 3.1rem;
  border-left: 2px solid #546e7a;
}

.ds-arrow-line::after {
  content: '';
  position: absolute;
  left: -6px;
  bottom: -2px;
  border-left: 5px solid transparent;
  border-right: 5px solid transparent;
  border-top: 9px solid #546e7a;
}
</style>
