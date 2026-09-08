<script setup lang="ts">
import { useNav, useSlideContext } from '@slidev/client'
import { computed, onUnmounted, ref, watch } from 'vue'

const P = 11
const MAX_K = 8
const STEP_MS = 750
const SIGMA = 1.6
const PEAK = 3

const MINI_W = 48
const MINI_H = 40
const SUM_W = 260
const SUM_H = 88
const LABEL_H = 16
const MINI_GAP = 1
const SUM_GAP = 2.5

function makeBaseDist(): Float64Array {
  const raw = Float64Array.from({ length: P }, (_, i) => {
    const d = Math.min(Math.abs(i - PEAK), P - Math.abs(i - PEAK))
    return Math.exp(-(d * d) / (2 * SIGMA * SIGMA))
  })
  const s = raw.reduce((a, b) => a + b, 0)
  return Float64Array.from(raw, x => x / s)
}

function convolveModP(a: Float64Array, b: Float64Array): Float64Array {
  const out = new Float64Array(P).fill(0)
  for (let i = 0; i < P; i++) {
    for (let j = 0; j < P; j++)
      out[(i + j) % P] += a[i]! * b[j]!
  }
  return out
}

const baseDist = makeBaseDist()
const sumDists: Float64Array[] = [baseDist]
for (let k = 2; k <= MAX_K; k++)
  sumDists.push(convolveModP(sumDists[k - 2]!, baseDist))

const uniformProb = 1 / P
const yMax = Math.max(...baseDist, uniformProb * 1.05)

function barRects(
  dist: Float64Array,
  width: number,
  height: number,
  gap: number,
  accent: string,
) {
  const innerW = width - gap * (P - 1)
  const barW = innerW / P
  return Array.from({ length: P }, (_, i) => ({
    x: i * (barW + gap),
    y: height - (dist[i]! / yMax) * height,
    w: barW,
    h: (dist[i]! / yMax) * height,
    fill: accent,
    label: String(i),
  }))
}

const miniBars = computed(() =>
  barRects(baseDist, MINI_W, MINI_H, MINI_GAP, '#e65100'),
)

const { $clicks } = useSlideContext()
const { isPrintMode } = useNav()

const currentK = ref(0)
const animating = ref(false)

let token = 0
let sequenceStarted = false

/** click=0 では非表示、click>=1 で表示＋アニメ開始 */
const revealed = computed(() => isPrintMode.value || ($clicks.value ?? 0) >= 1)

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
  currentK.value = 0
  animating.value = false
}

function applyPrintFinalState() {
  token += 1
  sequenceStarted = true
  currentK.value = MAX_K
  animating.value = false
}

async function runSequence() {
  if (isPrintMode.value) {
    applyPrintFinalState()
    return
  }

  const t = ++token
  currentK.value = 0
  animating.value = true

  await delay(300, t)
  if (t !== token)
    return

  for (let k = 1; k <= MAX_K; k++) {
    currentK.value = k
    await delay(STEP_MS, t)
    if (t !== token)
      return
  }

  animating.value = false
}

const sumBars = computed(() => {
  if (currentK.value < 1)
    return []
  const dist = sumDists[currentK.value - 1]!
  return barRects(dist, SUM_W, SUM_H, SUM_GAP, '#ef6c00')
})

const uniformY = computed(() => SUM_H - (uniformProb / yMax) * SUM_H)

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
  <div class="fsu-root">
    <!-- click ステップを登録しつつ、click=0 では描画しない -->
    <div v-click class="fsu-click-slot" aria-hidden="true" />
    <div
      v-if="revealed"
      class="fsu"
      :class="{ 'is-live': currentK > 0 }"
    >
      <div class="fsu-split">
        <!-- 左: 各 X_i の分布 -->
        <div class="fsu-panel fsu-panel--left">
          <div class="fsu-panel-head">
            i.i.d. な <Math tex="\mathbb{F}_p" /> 値確率変数
          </div>
          <div class="fsu-panel-sub">
            各 <Math tex="X_i" /> は低エントロピー
          </div>

          <div class="fsu-vars-grid">
            <div
              v-for="i in currentK"
              :key="`var-${i}`"
              class="fsu-var"
              :class="{ 'is-new': i === currentK && animating }"
            >
              <svg
                class="fsu-mini-chart"
                :viewBox="`0 0 ${MINI_W} ${MINI_H}`"
                aria-hidden="true"
              >
                <rect
                  v-for="(bar, bi) in miniBars"
                  :key="`mb-${i}-${bi}`"
                  :x="bar.x"
                  :y="bar.y"
                  :width="bar.w"
                  :height="bar.h"
                  :fill="bar.fill"
                  opacity="0.85"
                  rx="0.6"
                />
              </svg>
              <span class="fsu-var-label">
                <Math :tex="`X_{${i}}`" />
              </span>
            </div>
          </div>
        </div>

        <!-- 中央: 和への矢印 -->
        <div class="fsu-bridge" :class="{ 'is-on': currentK > 0 }">
          <span class="fsu-bridge-plus">+</span>
          <span class="fsu-bridge-mod">mod <Math tex="p" /></span>
          <span class="fsu-bridge-arrow">→</span>
        </div>

        <!-- 右: 和の分布 -->
        <div class="fsu-panel fsu-panel--right" :class="{ 'is-on': currentK > 0 }">
          <div class="fsu-panel-head">
            <Math tex="X_1 + \cdots + X_k" /> の分布
          </div>
          <div class="fsu-k-badge">
            <Math tex="k" /> = {{ currentK || '…' }}
          </div>

          <svg
            v-if="currentK > 0"
            class="fsu-sum-chart"
            :viewBox="`0 0 ${SUM_W} ${SUM_H + LABEL_H}`"
            role="img"
            aria-label="Sum distribution mod p"
          >
            <line
              :x1="0"
              :y1="uniformY"
              :x2="SUM_W"
              :y2="uniformY"
              class="fsu-uniform-line"
            />
            <text
              :x="SUM_W - 2"
              :y="uniformY - 4"
              class="fsu-uniform-label"
              text-anchor="end"
            >
              一様 ≈ {{ (uniformProb * 100).toFixed(1) }}%
            </text>

            <g class="fsu-sum-bars">
              <rect
                v-for="(bar, bi) in sumBars"
                :key="`sb-${currentK}-${bi}`"
                :x="bar.x"
                :y="bar.y"
                :width="bar.w"
                :height="bar.h"
                :fill="bar.fill"
                rx="1.4"
                class="fsu-sum-bar"
              />
            </g>

            <g class="fsu-x-labels">
              <text
                v-for="(bar, bi) in sumBars"
                :key="`xl-${bi}`"
                :x="bar.x + bar.w / 2"
                :y="SUM_H + 11"
                class="fsu-x-label"
                text-anchor="middle"
              >
                {{ bar.label }}
              </text>
            </g>
          </svg>

          <div
            v-if="currentK > 0"
            class="fsu-status"
            :class="{ 'is-uniform': currentK >= MAX_K - 1 }"
          >
            <template v-if="currentK < MAX_K - 1">
              まだ偏っている…
            </template>
            <template v-else>
              <Math tex="\mathbb{F}_p" /> 上の一様分布に近づく
            </template>
          </div>

          <div v-else class="fsu-placeholder">
            和を取ると…
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.fsu-root {
  width: 100%;
  padding: 0 0.2rem;
  box-sizing: border-box;
}

.fsu-click-slot {
  position: absolute;
  width: 0;
  height: 0;
  overflow: hidden;
  pointer-events: none;
}

.fsu {
  margin-top: 0.15rem;
  width: 100%;
}

.fsu-split {
  display: flex;
  align-items: stretch;
  gap: 0.5rem;
  width: 100%;
  min-height: 9.5rem;
}

.fsu-panel {
  flex: 1 1 0;
  min-width: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.3rem;
  padding: 0.45rem 0.55rem 0.5rem;
  border-radius: 10px;
  border: 1.5px solid rgba(239, 108, 0, 0.35);
  background: linear-gradient(160deg, #fffdf8 0%, #fff8ee 100%);
}

.fsu-panel--right {
  border-color: rgba(239, 108, 0, 0.5);
  background: linear-gradient(160deg, #fffaf3 0%, #fff3e0 100%);
  opacity: 0.45;
  transition: opacity 0.35s ease;
}

.fsu-panel--right.is-on {
  opacity: 1;
}

.fsu-panel-head {
  font-size: 0.76rem;
  font-weight: 700;
  color: #bf360c;
  text-align: center;
  line-height: 1.3;
}

.fsu-panel-head :deep(.katex) {
  font-size: 1em;
}

.fsu-panel-sub {
  font-size: 0.68rem;
  font-weight: 600;
  color: #78909c;
  text-align: center;
  line-height: 1.2;
  margin-top: -0.15rem;
}

.fsu-panel-sub :deep(.katex) {
  font-size: 1em;
}

.fsu-vars-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-template-rows: repeat(2, auto);
  gap: 0.32rem 0.28rem;
  width: 100%;
  flex: 1 1 auto;
  padding: 0.15rem 0.05rem 0;
  justify-items: center;
  align-content: start;
}

.fsu-var {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.12rem;
  flex: 0 0 auto;
  animation: fsu-var-in 0.4s cubic-bezier(0.22, 1, 0.36, 1) both;
}

.fsu-var.is-new {
  animation: fsu-var-pop 0.45s cubic-bezier(0.22, 1, 0.36, 1) both;
}

@keyframes fsu-var-in {
  from {
    opacity: 0;
    transform: scale(0.82) translateY(4px);
  }
  to {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}

@keyframes fsu-var-pop {
  0% {
    opacity: 0;
    transform: scale(0.7) translateY(6px);
  }
  70% {
    transform: scale(1.06) translateY(-1px);
  }
  100% {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}

.fsu-mini-chart {
  width: 100%;
  max-width: 2.85rem;
  height: 2.35rem;
  border-radius: 6px;
  background: #fff;
  border: 1px solid rgba(230, 81, 0, 0.28);
  padding: 0.1rem;
  box-sizing: border-box;
}

.fsu-var-label {
  font-size: 0.72rem;
  font-weight: 700;
  color: #e65100;
}

.fsu-var-label :deep(.katex) {
  font-size: 1em;
}

.fsu-bridge {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 0.2rem;
  flex: 0 0 2.2rem;
  align-self: center;
  opacity: 0.25;
  transition: opacity 0.35s ease;
}

.fsu-bridge.is-on {
  opacity: 1;
}

.fsu-bridge-plus {
  font-size: 1.35rem;
  font-weight: 800;
  color: #90a4ae;
  line-height: 1;
}

.fsu-bridge-mod {
  font-size: 0.62rem;
  font-weight: 700;
  color: #78909c;
  white-space: nowrap;
}

.fsu-bridge-mod :deep(.katex) {
  font-size: 1em;
}

.fsu-bridge-arrow {
  font-size: 1.5rem;
  font-weight: 700;
  color: #ef6c00;
  line-height: 1;
}

.fsu-k-badge {
  display: inline-flex;
  align-items: baseline;
  gap: 0.15em;
  padding: 0.15rem 0.5rem;
  border-radius: 999px;
  background: #fff;
  border: 1px solid rgba(239, 108, 0, 0.4);
  font-size: 0.74rem;
  font-weight: 800;
  color: #e65100;
}

.fsu-k-badge :deep(.katex) {
  font-size: 1em;
}

.fsu-sum-chart {
  display: block;
  width: 100%;
  max-width: 16rem;
  height: auto;
  overflow: visible;
}

.fsu-uniform-line {
  stroke: #43a047;
  stroke-width: 1.6;
  stroke-dasharray: 5 3;
  opacity: 0.9;
}

.fsu-uniform-label {
  font-size: 10px;
  font-weight: 700;
  fill: #2e7d32;
}

.fsu-sum-bar {
  transition: y 0.55s cubic-bezier(0.22, 1, 0.36, 1),
    height 0.55s cubic-bezier(0.22, 1, 0.36, 1);
}

.fsu-x-label {
  font-size: 10px;
  font-weight: 600;
  fill: #78909c;
}

.fsu-status {
  font-size: 0.74rem;
  font-weight: 700;
  color: #90a4ae;
  text-align: center;
  line-height: 1.35;
  transition: color 0.35s ease;
}

.fsu-status.is-uniform {
  color: #2e7d32;
}

.fsu-status :deep(.katex) {
  font-size: 1em;
}

.fsu-placeholder {
  flex: 1 1 auto;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.72rem;
  font-weight: 600;
  color: #b0bec5;
  font-style: italic;
}
</style>
