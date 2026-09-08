<script setup lang="ts">
import { useNav, useSlideContext } from '@slidev/client'
import { computed, onUnmounted, ref, watch } from 'vue'
import MathTex from './Math.vue'

const OUTER = { x: 50, y: 10, w: 220, h: 110 }
const ELLIPSE = { cx: 160, cy: 65, rx: 30, ry: 30 }
const POINT_COUNT = 6
const STEP_MS = 750
const HIGHLIGHT_DELAY_MS = 400

const RAW_POINTS = [
  { x: 88, y: 42 },
  { x: 215, y: 88 },
  { x: 163, y: 67 },
  { x: 118, y: 98 },
  { x: 235, y: 38 },
  { x: 72, y: 78 },
] as const

const F_VALUES = [0, 1, 1, 0, 1, 0] as const

function inEllipse(x: number, y: number) {
  const dx = (x - ELLIPSE.cx) / ELLIPSE.rx
  const dy = (y - ELLIPSE.cy) / ELLIPSE.ry
  return dx * dx + dy * dy <= 1
}

const points = RAW_POINTS.map((p, i) => ({
  id: i + 1,
  x: p.x,
  y: p.y,
  inH: inEllipse(p.x, p.y),
  fValue: F_VALUES[i] ?? 0,
}))

const { $clicks } = useSlideContext()
const { isPrintMode } = useNav()

const visibleCount = ref(0)
const highlighted = ref(false)
const animating = ref(false)

let token = 0
let sequenceStarted = false

const revealed = computed(() => isPrintMode.value || ($clicks.value ?? 0) >= 1)
const hasPointInH = computed(() => points.some(p => p.inH))

const formulaTex = computed(() => {
  if (visibleCount.value === 0)
    return ''

  const terms: string[] = []
  for (let i = 0; i < visibleCount.value; i++) {
    if (i > 0)
      terms.push('\\oplus')
    const point = points[i]!
    const label = `f(x_{${point.id}})`
    if (highlighted.value && point.inH)
      terms.push(`\\textcolor{#b85450}{${label}}`)
    else
      terms.push(label)
  }

  if (highlighted.value && visibleCount.value === POINT_COUNT) {
    if (hasPointInH.value)
      terms.push('\\approx \\text{ランダムビット}')
    else
      terms.push('\\text{（計算可能）}')
  }

  return terms.join(' ')
})

function delay(ms: number, t: number) {
  return new Promise<void>((resolve) => {
    window.setTimeout(() => {
      if (t === token)
        resolve()
    }, ms)
  })
}

function resetAll() {
  token += 1
  sequenceStarted = false
  visibleCount.value = 0
  highlighted.value = false
  animating.value = false
}

function applyPrintFinalState() {
  token += 1
  sequenceStarted = true
  visibleCount.value = POINT_COUNT
  highlighted.value = true
  animating.value = false
}

async function runSequence() {
  if (isPrintMode.value) {
    applyPrintFinalState()
    return
  }

  const t = ++token
  visibleCount.value = 0
  highlighted.value = false
  animating.value = true

  await delay(250, t)
  if (t !== token)
    return

  for (let i = 1; i <= POINT_COUNT; i++) {
    visibleCount.value = i
    await delay(STEP_MS, t)
    if (t !== token)
      return
  }

  await delay(HIGHLIGHT_DELAY_MS, t)
  if (t !== token)
    return

  highlighted.value = true
  animating.value = false
}

watch(
  [$clicks, isPrintMode],
  ([current]) => {
    if (isPrintMode.value) {
      applyPrintFinalState()
      return
    }

    if ((current ?? 0) < 1) {
      resetAll()
      return
    }

    if (!sequenceStarted) {
      sequenceStarted = true
      runSequence()
    }
  },
  { immediate: true },
)

onUnmounted(() => {
  resetAll()
})
</script>

<template>
  <div class="xor-hardcore-root">
    <div v-click class="xor-hardcore-click-slot" aria-hidden="true" />

    <div v-if="revealed" class="xor-hardcore">
      <svg
        viewBox="0 0 272 122"
        class="xor-hardcore-svg"
        role="img"
        aria-label="ハードコア集合への独立サンプル"
      >
        <rect
          x="50"
          y="10"
          width="220"
          height="110"
          rx="16.5"
          ry="16.5"
          class="xor-hardcore-domain"
        />

        <ellipse
          cx="160"
          cy="65"
          rx="30"
          ry="30"
          class="xor-hardcore-set"
        />

        <g v-for="point in points.slice(0, visibleCount)" :key="point.id">
          <circle
            :cx="point.x"
            :cy="point.y"
            r="4.5"
            class="xor-hardcore-point"
            :class="{
              'is-new': animating && point.id === visibleCount,
              'is-in-h': highlighted && point.inH,
            }"
          />
          <foreignObject
            :x="point.x + 4"
            :y="point.y - 20"
            width="28"
            height="18"
            class="xor-hardcore-point-label-wrap"
          >
            <div
              xmlns="http://www.w3.org/1999/xhtml"
              class="xor-hardcore-point-label"
              :class="{ 'is-in-h': highlighted && point.inH }"
            >
              <MathTex :tex="`x_{${point.id}}`" />
            </div>
          </foreignObject>
        </g>

        <foreignObject x="170" y="20" width="60" height="30">
          <div xmlns="http://www.w3.org/1999/xhtml" class="xor-hardcore-label">
            <MathTex tex="H" />
          </div>
        </foreignObject>
      </svg>

      <div class="xor-hardcore-formula" :class="{ 'has-content': visibleCount > 0 }">
        <MathTex v-if="formulaTex" :tex="formulaTex" :display="true" />
      </div>

      <p v-if="highlighted && hasPointInH" class="xor-hardcore-note">
        ある <MathTex tex="x_i \in H" /> なら <MathTex tex="f(x_i)" /> はランダムビットのように見える
      </p>
    </div>
  </div>
</template>

<style scoped>
.xor-hardcore-root {
  width: 100%;
}

.xor-hardcore-click-slot {
  position: absolute;
  width: 0;
  height: 0;
  overflow: hidden;
  pointer-events: none;
}

.xor-hardcore {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.35rem;
  margin-top: 0.2rem;
}

.xor-hardcore-svg {
  display: block;
  width: 100%;
  max-width: 22rem;
  height: auto;
  overflow: visible;
}

.xor-hardcore-domain {
  fill: #fff;
  stroke: #000;
}

.xor-hardcore-set {
  fill: #f8cecc;
  stroke: #b85450;
  stroke-width: 1.5;
}

.xor-hardcore-label {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
  font-size: 0.95rem;
  color: #263238;
  pointer-events: none;
}

.xor-hardcore-point-label-wrap {
  overflow: visible;
  pointer-events: none;
}

.xor-hardcore-point-label {
  display: flex;
  align-items: center;
  justify-content: flex-start;
  width: 100%;
  height: 100%;
  line-height: 1;
  transition: color 0.35s ease;
}

.xor-hardcore-point-label :deep(.katex) {
  font-size: 0.62rem;
}

.xor-hardcore-point-label.is-in-h :deep(.katex) {
  color: #b85450;
}

.xor-hardcore-point {
  fill: #1976d2;
  stroke: #fff;
  stroke-width: 1.2;
  transition: fill 0.35s ease, stroke 0.35s ease;
}

.xor-hardcore-point.is-new {
  animation: xor-point-pop 0.45s ease-out;
}

.xor-hardcore-point.is-in-h {
  fill: #b85450;
  stroke: #fff3e0;
}

.xor-hardcore-formula {
  min-height: 2.2rem;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  max-width: 36rem;
  text-align: center;
}

.xor-hardcore-formula.has-content :deep(.katex-display) {
  margin: 0.15em 0;
}

.xor-hardcore-note {
  margin: 0.1rem 0 0;
  font-size: 0.88rem;
  color: #546e7a;
  text-align: center;
}

@keyframes xor-point-pop {
  0% {
    opacity: 0;
  }

  100% {
    opacity: 1;
  }
}
</style>
