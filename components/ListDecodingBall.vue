<script setup lang="ts">
import { computed, onUnmounted, ref } from 'vue'
import MathTex from './Math.vue'

const props = withDefaults(defineProps<{
  unique?: boolean
  packing?: boolean
}>(), {
  unique: false,
  packing: false,
})

const DOMAIN = { w: 240, h: 168, rx: 14 }
const R_MIN = 12
const R_MAX = 96
const RIM_HIT = 8
const HANDLE_HIT = 9
const POINT_R = computed(() => props.unique ? 3.15 : 1.85)

function latticePoints(
  origin: { x: number, y: number },
  v1: { x: number, y: number },
  v2: { x: number, y: number },
  iMin: number,
  iMax: number,
  jMin: number,
  jMax: number,
  pad = 16,
) {
  const pts: { x: number, y: number }[] = []
  for (let i = iMin; i <= iMax; i++) {
    for (let j = jMin; j <= jMax; j++) {
      const x = origin.x + i * v1.x + j * v2.x
      const y = origin.y + i * v1.y + j * v2.y
      if (x < pad || x > DOMAIN.w - pad || y < pad || y > DOMAIN.h - pad)
        continue
      pts.push({ x, y })
    }
  }
  return pts
}

function makePoints() {
  if (props.unique) {
    return latticePoints(
      { x: 120, y: 86 },
      { x: 64, y: 17 },
      { x: -30, y: 48 },
      -1, 1, -1, 1,
    )
  }
  return latticePoints(
    { x: 120, y: 86 },
    { x: 32, y: 8.5 },
    { x: -15, y: 24 },
    -3, 3, -3, 3,
  )
}

const POINTS = makePoints()

const packingR = (() => {
  let d = Infinity
  for (let i = 0; i < POINTS.length; i++) {
    for (let j = i + 1; j < POINTS.length; j++) {
      const a = POINTS[i]!
      const b = POINTS[j]!
      d = Math.min(d, Math.hypot(a.x - b.x, a.y - b.y))
    }
  }
  return d / 2
})()

const rMax = props.packing ? Math.min(R_MAX, packingR * 1.9) : R_MAX

const cx = ref(120)
const cy = ref(86)
const ballR = ref(props.packing ? packingR : props.unique ? 32 : 44)
const dragging = ref(false)
const resizing = ref(false)
const overRim = ref(false)

let dragOffsetX = 0
let dragOffsetY = 0
let activePointer: number | null = null

function distToCenter(x: number, y: number) {
  return Math.hypot(x - cx.value, y - cy.value)
}

function inBall(x: number, y: number) {
  return distToCenter(x, y) <= ballR.value
}

function nearRim(x: number, y: number) {
  const hx = cx.value + ballR.value
  const hy = cy.value
  if (Math.hypot(x - hx, y - hy) <= HANDLE_HIT)
    return true
  return Math.abs(distToCenter(x, y) - ballR.value) <= RIM_HIT
}

function clampRadius(r: number) {
  return Math.min(rMax, Math.max(R_MIN, r))
}

function onRadiusInput(event: Event) {
  const el = event.target as HTMLInputElement
  ballR.value = clampRadius(Number(el.value))
}

const overlapping = computed(() => props.packing && ballR.value > packingR + 0.15)

const count = computed(() => POINTS.filter(p => inBall(p.x, p.y)).length)

const countTex = computed(() =>
  `\\lvert\\mathrm{ball}(y,\\rho)\\cap\\mathcal{C}\\rvert = ${count.value}`,
)

function svgCoords(svg: SVGSVGElement, event: PointerEvent) {
  const pt = svg.createSVGPoint()
  pt.x = event.clientX
  pt.y = event.clientY
  const matrix = svg.getScreenCTM()
  if (!matrix)
    return { x: cx.value, y: cy.value }
  return pt.matrixTransform(matrix.inverse())
}

function clampCenter(x: number, y: number) {
  return {
    x: Math.min(DOMAIN.w, Math.max(0, x)),
    y: Math.min(DOMAIN.h, Math.max(0, y)),
  }
}

function onPointerDown(event: PointerEvent) {
  if (props.packing)
    return
  if (event.button !== 0 && event.pointerType === 'mouse')
    return
  event.preventDefault()
  event.stopPropagation()
  const svg = event.currentTarget as SVGSVGElement
  const p = svgCoords(svg, event)
  const resize = nearRim(p.x, p.y)
  if (!resize && !inBall(p.x, p.y))
    return
  activePointer = event.pointerId
  if (resize) {
    resizing.value = true
    dragging.value = false
    ballR.value = clampRadius(distToCenter(p.x, p.y))
  }
  else {
    dragging.value = true
    resizing.value = false
    dragOffsetX = cx.value - p.x
    dragOffsetY = cy.value - p.y
  }
  svg.setPointerCapture(event.pointerId)
}

function onPointerMove(event: PointerEvent) {
  if (props.packing)
    return
  const svg = event.currentTarget as SVGSVGElement
  const p = svgCoords(svg, event)
  if (activePointer === event.pointerId) {
    event.preventDefault()
    event.stopPropagation()
    if (resizing.value) {
      ballR.value = clampRadius(distToCenter(p.x, p.y))
      return
    }
    if (dragging.value) {
      const next = clampCenter(p.x + dragOffsetX, p.y + dragOffsetY)
      cx.value = next.x
      cy.value = next.y
      return
    }
  }
  overRim.value = nearRim(p.x, p.y)
}

function endDrag(event: PointerEvent) {
  if (event.pointerId !== activePointer)
    return
  dragging.value = false
  resizing.value = false
  activePointer = null
  const svg = event.currentTarget as SVGSVGElement
  if (svg.hasPointerCapture(event.pointerId))
    svg.releasePointerCapture(event.pointerId)
  const p = svgCoords(svg, event)
  overRim.value = nearRim(p.x, p.y)
}

function onPointerLeave() {
  if (activePointer !== null)
    return
  overRim.value = false
}

onUnmounted(() => {
  dragging.value = false
  resizing.value = false
  activePointer = null
})
</script>

<template>
  <div class="ld" :class="{ 'is-packing': packing, 'is-compact': !unique }" @click.stop>
    <svg
      class="ld-svg"
      :class="{ 'is-dragging': dragging, 'is-resizing': resizing, 'is-over-rim': overRim }"
      viewBox="-12 -12 264 192"
      role="img"
      :aria-label="packing ? '符号語を中心とする一意復号半径のハミングボール' : unique ? '一意復号のハミングボール' : 'リスト復号のハミングボール'"
      @pointerdown="onPointerDown"
      @pointermove="onPointerMove"
      @pointerup="endDrag"
      @pointercancel="endDrag"
      @pointerleave="onPointerLeave"
    >
      <rect
        :x="0"
        :y="0"
        :width="DOMAIN.w"
        :height="DOMAIN.h"
        :rx="DOMAIN.rx"
        :ry="DOMAIN.rx"
        class="ld-domain"
      />

      <template v-if="packing">
        <circle
          v-for="(p, i) in POINTS"
          :key="`pack-${i}`"
          :cx="p.x"
          :cy="p.y"
          :r="ballR"
          class="ld-pack"
          :class="{ 'is-overlap': overlapping }"
        />
      </template>

      <template v-if="!packing">
      <circle
        :cx="cx"
        :cy="cy"
        :r="ballR"
        class="ld-ball"
        :class="{ 'is-dragging': dragging, 'is-resizing': resizing }"
      />

      <circle
        :cx="cx"
        :cy="cy"
        :r="ballR"
        class="ld-rim-hit"
      />

      <circle
        :cx="cx + ballR"
        :cy="cy"
        r="4.4"
        class="ld-handle"
        :class="{ 'is-active': resizing || overRim }"
      />
      </template>

      <circle
        v-for="(p, i) in POINTS"
        :key="i"
        :cx="p.x"
        :cy="p.y"
        :r="POINT_R"
        class="ld-point"
        :class="{ 'is-in': !packing && inBall(p.x, p.y), 'is-code': packing }"
      />

      <template v-if="!packing">
      <circle
        :cx="cx"
        :cy="cy"
        :r="2.5"
        class="ld-center"
      />

      <foreignObject
        :x="cx - 10"
        :y="cy + 5"
        width="20"
        height="18"
      >
        <div xmlns="http://www.w3.org/1999/xhtml" class="ld-ylabel">
          <MathTex tex="y" />
        </div>
      </foreignObject>
      </template>
    </svg>

    <div class="ld-side">
      <div
        class="ld-knob"
        @click.stop
        @pointerdown.stop
        @pointerup.stop
        @keydown.stop
      >
        <span class="ld-knob-label"><MathTex tex="\rho" /></span>
        <div class="ld-knob-track">
          <input
            type="range"
            :min="R_MIN"
            :max="rMax"
            step="0.25"
            :value="ballR"
            aria-label="ハミングボールの半径"
            @input="onRadiusInput"
          >
        </div>
        <button
          v-if="packing"
          type="button"
          class="ld-knob-hint"
          @click="ballR = packingR"
        >
          <MathTex tex="\delta/2" />
        </button>
      </div>

      <div v-if="!packing" class="ld-count">
        <MathTex :tex="countTex" />
      </div>
    </div>
  </div>
</template>

<style scoped>
.ld {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1.15rem;
  margin: 0.25rem auto 0;
  user-select: none;
  touch-action: none;
}

.ld-svg {
  display: block;
  width: 24.5rem;
  max-width: calc(100% - 8.5rem);
  height: auto;
  overflow: visible;
  cursor: default;
  touch-action: none;
}

.ld.is-packing .ld-svg {
  max-width: calc(100% - 4.2rem);
}

.ld.is-compact .ld-svg {
  width: 17.5rem;
}

.ld-side {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 0.85rem;
  flex: 0 0 auto;
}

.ld-knob {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.2rem;
}

.ld-knob-label {
  font-size: 1.05rem;
  color: #ff8000;
  line-height: 1;
}

.ld-knob-hint {
  font-size: 0.78rem;
  color: #78909c;
  line-height: 1;
  padding: 0.15rem 0.2rem;
  border: none;
  background: transparent;
  cursor: pointer;
}

.ld-knob-hint:hover {
  color: #ff8000;
}

.ld-knob-track {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 2.1rem;
  height: 8.6rem;
}

.ld-knob-track input {
  width: 8.6rem;
  height: 1.35rem;
  margin: 0;
  transform: rotate(-90deg);
  accent-color: #ff8000;
  cursor: grab;
  touch-action: none;
}

.ld-knob-track input:active {
  cursor: grabbing;
}

.ld.is-compact .ld-knob-track {
  height: 6.5rem;
}

.ld.is-compact .ld-knob-track input {
  width: 6.5rem;
}

.ld-svg.is-dragging {
  cursor: grabbing;
}

.ld-svg.is-over-rim,
.ld-svg.is-resizing {
  cursor: ew-resize;
}

.ld-domain {
  fill: #fff;
  stroke: #263238;
  stroke-width: 0.75;
}

.ld-pack {
  fill: rgba(255, 128, 0, 0.16);
  stroke: #ff8000;
  stroke-width: 1.05;
  pointer-events: none;
}

.ld-pack.is-overlap {
  fill: rgba(194, 24, 91, 0.14);
  stroke: #c2185b;
}

.ld-ball {
  fill: rgba(255, 128, 0, 0.14);
  stroke: #ff8000;
  stroke-width: 1.15;
  cursor: grab;
}

.ld-ball.is-dragging,
.ld-ball.is-resizing {
  fill: rgba(255, 128, 0, 0.22);
}

.ld-rim-hit {
  fill: none;
  stroke: transparent;
  stroke-width: 12;
  pointer-events: none;
}

.ld-handle {
  fill: #fff;
  stroke: #ff8000;
  stroke-width: 1.15;
  pointer-events: none;
}

.ld-handle.is-active {
  fill: #ff8000;
}

.ld-point {
  fill: #212121;
  stroke: none;
  pointer-events: none;
  transition: fill 0.12s ease;
}

.ld-point.is-in,
.ld-point.is-code {
  fill: #c2185b;
}

.ld-center {
  fill: #b85450;
  stroke: none;
  pointer-events: none;
}

.ld-ylabel {
  display: flex;
  align-items: flex-start;
  justify-content: center;
  width: 100%;
  height: 100%;
  font-size: 0.72rem;
  color: #b85450;
  pointer-events: none;
}

.ld-count {
  flex: 0 0 auto;
  min-width: 8.4rem;
  font-size: 1.12rem;
  font-weight: 600;
  color: #37474f;
  white-space: nowrap;
}

.ld-count :deep(.math) {
  font-size: 1.05em;
}
</style>
