<script setup lang="ts">
import { computed, onUnmounted, ref, watch } from 'vue'
import { useNav, useSlideContext } from '@slidev/client'
import MathTex from './Math.vue'

const HIST = [
  { year: 1968, omega: 3 },
  { year: 1968, omega: 2.807 },
  { year: 1978, omega: 2.795 },
  { year: 1979, omega: 2.779 },
  { year: 1981, omega: 2.522 },
  { year: 1981, omega: 2.517 },
  { year: 1981, omega: 2.496 },
  { year: 1986, omega: 2.479 },
  { year: 1990, omega: 2.3755 },
  { year: 2010, omega: 2.3737 },
  { year: 2012, omega: 2.3729 },
  { year: 2014, omega: 2.3728639 },
  { year: 2020, omega: 2.3728596 },
  { year: 2022, omega: 2.371866 },
  { year: 2024, omega: 2.371552 },
  { year: 2025, omega: 2.371339 },
] as const

const YEAR_MIN = 1968
const LAST_DATA_YEAR = 2025
const LAST_DATA_OMEGA = 2.371339
const OMEGA_1990 = 2.3755
const NOW_YEAR = 2026
const END_OMEGA = 2
const RATE = (OMEGA_1990 - LAST_DATA_OMEGA) / (LAST_DATA_YEAR - 1990)
const NOW_OMEGA = LAST_DATA_OMEGA - RATE * (NOW_YEAR - LAST_DATA_YEAR)
const YEARS_TO_TWO = (NOW_OMEGA - END_OMEGA) / RATE
const END_YEAR = NOW_YEAR + YEARS_TO_TWO
const DISPLAY_DELTA = 0.00416
const DISPLAY_YEARS = Math.round(YEARS_TO_TWO)
const DISPLAY_YEAR = Math.round(END_YEAR)
const INIT_XMAX = NOW_YEAR
const DURATION_MS = 4200
const MIN_TICK_PX = 52

const W = 720
const H = 338
const PAD = { l: 58, r: 48, t: 18, b: 46 }
const X0 = PAD.l
const X1 = W - PAD.r
const Y0 = PAD.t
const Y1 = H - PAD.b
const AXIS_TITLE_Y = (Y0 + Y1) / 2
const OMEGA_MAX = 3
const OMEGA_MIN = 2

const { $clicks } = useSlideContext()
const { isPrintMode } = useNav()

const progress = ref(0)
const playing = ref(false)

let raf = 0
let startAt = 0

const revealed = computed(() => isPrintMode.value || ($clicks.value ?? 0) >= 1)
const done = computed(() => progress.value >= 1)

const xMax = computed(() => INIT_XMAX + progress.value * (END_YEAR - INIT_XMAX))
const currentYear = computed(() => Math.round(xMax.value))
const currentOmega = computed(() => omegaAt(xMax.value))

function easeInCubic(t: number) {
  return t * t * t
}

function omegaAt(year: number) {
  return LAST_DATA_OMEGA - RATE * (year - LAST_DATA_YEAR)
}

function yOf(omega: number) {
  return Y0 + ((OMEGA_MAX - omega) / (OMEGA_MAX - OMEGA_MIN)) * (Y1 - Y0)
}

function xOf(year: number, maxYear = xMax.value) {
  return X0 + (year - YEAR_MIN) / (maxYear - YEAR_MIN) * (X1 - X0)
}

const histPoints = computed(() => {
  const maxYear = xMax.value
  const pts = HIST.map(p => `${xOf(p.year, maxYear).toFixed(2)},${yOf(p.omega).toFixed(2)}`)
  if (maxYear >= NOW_YEAR)
    pts.push(`${xOf(NOW_YEAR, maxYear).toFixed(2)},${yOf(NOW_OMEGA).toFixed(2)}`)
  return pts.join(' ')
})

const extraPoints = computed(() => {
  if (progress.value <= 0)
    return ''
  const maxYear = xMax.value
  const xStart = xOf(NOW_YEAR, maxYear)
  const yStart = yOf(NOW_OMEGA)
  const x = xOf(maxYear, maxYear)
  const y = yOf(omegaAt(maxYear))
  return `${xStart.toFixed(2)},${yStart.toFixed(2)} ${x.toFixed(2)},${y.toFixed(2)}`
})

const tip = computed(() => ({
  x: X1,
  y: yOf(currentOmega.value),
}))

const yearTicks = computed(() => {
  const maxYear = xMax.value
  const candidates = [1968, 1978, 1981, 1986, 1990, 2010, 2020, 2026]
  const ticks: number[] = []
  for (const year of candidates) {
    if (year > maxYear - 12)
      continue
    const x = xOf(year, maxYear)
    const last = ticks[ticks.length - 1]
    if (last != null && x - xOf(last, maxYear) < MIN_TICK_PX)
      continue
    ticks.push(year)
  }
  return ticks
})

function cancel() {
  if (raf) {
    cancelAnimationFrame(raf)
    raf = 0
  }
}

function reset() {
  cancel()
  playing.value = false
  progress.value = 0
}

function tick(now: number) {
  const t = Math.min(1, (now - startAt) / DURATION_MS)
  progress.value = easeInCubic(t)
  if (t < 1) {
    raf = requestAnimationFrame(tick)
    return
  }
  progress.value = 1
  playing.value = false
  raf = 0
}

function start() {
  if (playing.value || progress.value >= 1)
    return
  playing.value = true
  startAt = performance.now()
  raf = requestAnimationFrame(tick)
}

watch(
  revealed,
  (on) => {
    if (isPrintMode.value) {
      progress.value = 1
      return
    }
    if (on)
      start()
    else
      reset()
  },
  { immediate: true },
)

onUnmounted(reset)
</script>

<template>
  <div class="omega-chart">
    <div v-click class="omega-click-slot" aria-hidden="true" />

    <svg
      class="omega-chart-svg"
      viewBox="0 0 720 338"
      role="img"
      aria-label="行列積の指数 omega の推移"
    >
      <line
        v-for="tick in [3, 2.8, 2.6, 2.4, 2.2, 2]"
        :key="tick"
        :x1="X0"
        :x2="X1"
        :y1="yOf(tick)"
        :y2="yOf(tick)"
        class="omega-grid"
        :class="{ 'omega-grid-target': tick === 2 }"
      />

      <line :x1="X0" :y1="Y0" :x2="X0" :y2="Y1" class="omega-axis" />
      <line :x1="X0" :y1="Y1" :x2="X1" :y2="Y1" class="omega-axis" />

      <text
        :x="18"
        :y="AXIS_TITLE_Y"
        class="omega-axis-title"
        :transform="`rotate(-90 18 ${AXIS_TITLE_Y})`"
      >ω</text>

      <text
        v-for="tick in [3, 2.8, 2.6, 2.4, 2.2, 2]"
        :key="`yl-${tick}`"
        :x="X0 - 8"
        :y="yOf(tick) + 4"
        class="omega-tick-label"
        text-anchor="end"
      >{{ tick.toFixed(1) }}</text>

      <text
        v-for="year in yearTicks"
        :key="`xl-${year}`"
        :x="xOf(year)"
        :y="Y1 + 18"
        class="omega-tick-label"
        text-anchor="middle"
      >{{ year }}</text>
      <text
        :x="X1"
        :y="Y1 + 18"
        class="omega-tick-label"
        :class="{ 'omega-tick-end': done }"
        text-anchor="end"
      >{{ currentYear }}</text>

      <polyline
        :points="histPoints"
        class="omega-hist"
        fill="none"
      />

      <circle
        :cx="xOf(NOW_YEAR)"
        :cy="yOf(NOW_OMEGA)"
        r="3.2"
        class="omega-now-dot"
      />

      <polyline
        v-if="extraPoints"
        :points="extraPoints"
        class="omega-future"
        fill="none"
      />

      <g v-if="progress > 0">
        <circle
          :cx="tip.x"
          :cy="tip.y"
          r="4.4"
          class="omega-tip"
          :class="{ 'is-done': done }"
        />
      </g>
    </svg>

    <p v-if="revealed && !done" class="omega-caption omega-caption-live">
      この35年で {{ DISPLAY_DELTA }} しか改善していない
    </p>
    <p v-else-if="done" class="omega-caption">
      同じ傾きが続くと,
      <MathTex tex="O(n^2)" />
      時間まであと {{ DISPLAY_YEARS }} 年（西暦{{ DISPLAY_YEAR }}年）.
    </p>
  </div>
</template>

<style scoped>
.omega-chart {
  position: relative;
  width: 92%;
  margin: 0 auto;
}

.omega-click-slot {
  position: absolute;
  width: 0;
  height: 0;
  overflow: hidden;
}

.omega-chart-svg {
  display: block;
  width: 100%;
  height: auto;
}

.omega-grid {
  stroke: #cfd8dc;
  stroke-width: 1;
}

.omega-grid-target {
  stroke: #90a4ae;
  stroke-dasharray: 4 4;
}

.omega-axis {
  stroke: #37474f;
  stroke-width: 1.4;
}

.omega-axis-title {
  fill: #37474f;
  font-size: 16px;
  font-weight: 600;
  font-family: 'Roboto', sans-serif;
}

.omega-tick-label {
  fill: #546e7a;
  font-size: 11px;
  font-family: 'Roboto', sans-serif;
}

.omega-hist {
  stroke: #1565c0;
  stroke-width: 2.4;
  stroke-linejoin: round;
  stroke-linecap: round;
}

.omega-future {
  stroke: #c2185b;
  stroke-width: 2.6;
  stroke-linecap: round;
}

.omega-now-dot {
  fill: #1565c0;
}

.omega-tip {
  fill: #c2185b;
}

.omega-tip.is-done {
  fill: #b71c1c;
}

.omega-tick-end {
  fill: #b71c1c;
  font-weight: 700;
}

.omega-caption {
  margin: 0.15rem 0 0;
  text-align: center;
  font-size: 0.92rem;
  color: #455a64;
  min-height: 1.6em;
}

.omega-caption-live {
  color: #c2185b;
  font-weight: 600;
}
</style>
