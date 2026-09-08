<script setup lang="ts">
import { ref } from 'vue'
import MathTex from './Math.vue'

const hoverState = ref<'hardcore' | 'easy' | null>(null)

const OUTER = { x: 50, y: 10, w: 220, h: 110 }
const ELLIPSE = { cx: 160, cy: 65, rx: 30, ry: 30 }

function svgPoint(svg: SVGSVGElement, event: MouseEvent) {
  const pt = svg.createSVGPoint()
  pt.x = event.clientX
  pt.y = event.clientY
  const matrix = svg.getScreenCTM()
  if (!matrix)
    return { x: 0, y: 0 }
  return pt.matrixTransform(matrix.inverse())
}

function inEllipse(x: number, y: number) {
  const dx = (x - ELLIPSE.cx) / ELLIPSE.rx
  const dy = (y - ELLIPSE.cy) / ELLIPSE.ry
  return dx * dx + dy * dy <= 1
}

function inOuter(x: number, y: number) {
  return x >= OUTER.x
    && x <= OUTER.x + OUTER.w
    && y >= OUTER.y
    && y <= OUTER.y + OUTER.h
}

function onMove(event: MouseEvent) {
  const svg = event.currentTarget as SVGSVGElement
  const { x, y } = svgPoint(svg, event)
  if (inEllipse(x, y))
    hoverState.value = 'hardcore'
  else if (inOuter(x, y))
    hoverState.value = 'easy'
  else
    hoverState.value = null
}

function onLeave() {
  hoverState.value = null
}
</script>

<template>
  <div class="hardcore-diagram">
    <svg
      viewBox="0 0 272 122"
      class="hardcore-diagram-svg"
      role="img"
      aria-label="ハードコア集合の模式図"
      @mousemove="onMove"
      @mouseleave="onLeave"
    >
      <rect
        x="50"
        y="10"
        width="220"
        height="110"
        rx="16.5"
        ry="16.5"
        class="hardcore-diagram-domain"
      />

      <ellipse
        cx="160"
        cy="65"
        rx="30"
        ry="30"
        class="hardcore-diagram-set"
        :class="{ 'is-active': hoverState === 'hardcore' }"
      />

      <foreignObject x="0" y="0" width="60" height="30">
        <div xmlns="http://www.w3.org/1999/xhtml" class="hardcore-diagram-label">
          <MathTex tex="\{0,1\}^n" />
        </div>
      </foreignObject>

      <foreignObject x="170" y="20" width="60" height="30">
        <div xmlns="http://www.w3.org/1999/xhtml" class="hardcore-diagram-label">
          <MathTex tex="H" />
        </div>
      </foreignObject>
    </svg>

    <p v-if="hoverState === 'hardcore'" class="hardcore-diagram-caption is-hardcore">
      <MathTex tex="f(x)=\text{ランダム}" />
    </p>
    <p v-else-if="hoverState === 'easy'" class="hardcore-diagram-caption is-easy">
      <MathTex tex="f(x)\text{は簡単に計算できる}" />
    </p>
  </div>
</template>

<style scoped>
.hardcore-diagram {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.45rem;
  margin: 0.35rem auto 0;
  max-width: 20rem;
}

.hardcore-diagram-svg {
  display: block;
  width: 100%;
  height: auto;
  overflow: visible;
}

.hardcore-diagram-domain {
  fill: #fff;
  stroke: #000;
  pointer-events: all;
}

.hardcore-diagram-set {
  fill: #f8cecc;
  stroke: #b85450;
  stroke-width: 1.5;
  pointer-events: all;
  transition: fill 0.15s ease, stroke 0.15s ease;
}

.hardcore-diagram-set.is-active {
  fill: #f4aba8;
  stroke: #a94440;
}

.hardcore-diagram-label {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
  font-size: 0.95rem;
  color: #263238;
  pointer-events: none;
}

.hardcore-diagram-caption {
  margin: 0;
  min-height: 1.6rem;
  font-size: 0.95rem;
  text-align: center;
  color: #37474f;
}

.hardcore-diagram-caption.is-hardcore {
  color: #b85450;
  font-weight: 600;
}
</style>
