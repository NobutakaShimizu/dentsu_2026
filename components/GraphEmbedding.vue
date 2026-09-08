<script setup lang="ts">
import { computed } from 'vue'
import { useSlideContext } from '@slidev/client'
import MathTex from './Math.vue'

type Point = { x: number, y: number }

const props = withDefaults(defineProps<{
  k?: number
}>(), {
  k: 4,
})

const { $clicks } = useSlideContext()
const step = computed(() => $clicks.value)

const showCrossEdges = computed(() => step.value >= 1)
const highlightAll = computed(() => step.value === 2)
const highlightCrossOnly = computed(() => step.value >= 3)
const edgesJustAdded = computed(() => step.value === 1)

const boxW = 176
const boxH = 160
const gap = 35
const startX = 20
const startY = 30
const labelOffsetY = 24
const labelBoxW = 44
const labelBoxH = 28

const graphs = Array.from({ length: props.k }, (_, i) => ({
  index: i + 1,
  x: startX + i * (boxW + gap),
  y: startY,
}))

function graphRect(i: number) {
  const x = startX + i * (boxW + gap)
  return { x, y: startY, right: x + boxW, left: x, top: startY, bottom: startY + boxH }
}

function inside(i: number, relX: number, relY: number): Point {
  const margin = 0.1
  const rx = Math.min(1 - margin, Math.max(margin, relX))
  const ry = Math.min(1 - margin, Math.max(margin, relY))
  const r = graphRect(i)
  return { x: r.x + rx * boxW, y: r.top + ry * boxH }
}

const crossEdges = [
  { gi: 0, xL: 0.85, yL: 0.22, xR: 0.15, yR: 0.28 },
  { gi: 0, xL: 0.88, yL: 0.52, xR: 0.12, yR: 0.48 },
  { gi: 1, xL: 0.86, yL: 0.28, xR: 0.14, yR: 0.18 },
  { gi: 1, xL: 0.84, yL: 0.72, xR: 0.16, yR: 0.55 },
  { gi: 2, xL: 0.87, yL: 0.24, xR: 0.13, yR: 0.30 },
  { gi: 2, xL: 0.85, yL: 0.68, xR: 0.15, yR: 0.52 },
].map(({ gi, xL, yL, xR, yR }) => {
  const a = inside(gi, xL, yL)
  const b = inside(gi + 1, xR, yR)
  const len = Math.hypot(b.x - a.x, b.y - a.y)
  return { x1: a.x, y1: a.y, x2: b.x, y2: b.y, a, b, len }
})

const crossTriangles = [
  { a: crossEdges[0].a, b: crossEdges[0].b, c: inside(0, 0.35, 0.25) },
  { a: crossEdges[2].a, b: crossEdges[2].b, c: inside(1, 0.45, 0.22) },
  { a: crossEdges[5].a, b: crossEdges[5].b, c: inside(3, 0.62, 0.65) },
]

// Two triangles per graph in opposite corners so dashed edges do not overlap.
const internalTriangles = [
  { a: inside(0, 0.18, 0.32), b: inside(0, 0.36, 0.16), c: inside(0, 0.24, 0.20) },
  { a: inside(0, 0.58, 0.68), b: inside(0, 0.80, 0.72), c: inside(0, 0.72, 0.52) },
  { a: inside(1, 0.64, 0.18), b: inside(1, 0.82, 0.32), c: inside(1, 0.70, 0.35) },
  { a: inside(1, 0.16, 0.58), b: inside(1, 0.34, 0.78), c: inside(1, 0.18, 0.72) },
  { a: inside(2, 0.16, 0.28), b: inside(2, 0.38, 0.16), c: inside(2, 0.20, 0.18) },
  { a: inside(2, 0.62, 0.70), b: inside(2, 0.84, 0.66), c: inside(2, 0.74, 0.52) },
  { a: inside(3, 0.58, 0.16), b: inside(3, 0.82, 0.24), c: inside(3, 0.68, 0.36) },
  { a: inside(3, 0.14, 0.56), b: inside(3, 0.36, 0.74), c: inside(3, 0.16, 0.68) },
]

const viewW = startX + props.k * boxW + (props.k - 1) * gap + 20
const viewH = startY + boxH + labelOffsetY + labelBoxH + 8

function graphIndexOf(p: Point): number {
  for (let i = 0; i < props.k; i++) {
    const r = graphRect(i)
    if (p.x >= r.x && p.x <= r.right && p.y >= r.top && p.y <= r.bottom)
      return i
  }
  return -1
}

function spansBoxes(p: Point, q: Point) {
  return graphIndexOf(p) !== graphIndexOf(q)
}

function pointsAttr(tri: typeof internalTriangles[number]) {
  return `${tri.a.x},${tri.a.y} ${tri.b.x},${tri.b.y} ${tri.c.x},${tri.c.y}`
}

function triangleEdges(tri: typeof internalTriangles[number]) {
  return [
    [tri.a, tri.b],
    [tri.b, tri.c],
    [tri.c, tri.a],
  ] as [Point, Point][]
}

function internalFillClass() {
  return {
    'is-highlighted': highlightAll.value,
    'is-dimmed': highlightCrossOnly.value,
    'is-no-fill': edgesJustAdded.value,
    'is-default-fill': step.value === 0,
  }
}

function crossFillClass() {
  return {
    'is-highlighted-sky': highlightAll.value || highlightCrossOnly.value,
    'is-no-fill': edgesJustAdded.value,
  }
}

function triCentroid(tri: typeof internalTriangles[number]) {
  return {
    x: (tri.a.x + tri.b.x + tri.c.x) / 3,
    y: (tri.a.y + tri.b.y + tri.c.y) / 3,
  }
}

function edgeMidpoint(x1: number, y1: number, x2: number, y2: number) {
  return { x: (x1 + x2) / 2, y: (y1 + y2) / 2 }
}
</script>

<template>
  <div class="graph-embedding">
    <svg :viewBox="`0 0 ${viewW} ${viewH}`" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Embedding of k subgraphs">
      <rect
        v-for="(g, i) in graphs"
        :key="g.index"
        :x="g.x"
        :y="g.y"
        :width="boxW"
        :height="boxH"
        rx="8"
        fill="#fff3e0"
        stroke="#ff9800"
        stroke-width="2.5"
        class="graph-box"
        :style="{ animationDelay: `${i * 0.1}s` }"
      />

      <polygon
        v-for="(tri, i) in internalTriangles"
        :key="`internal-fill-${i}`"
        :points="pointsAttr(tri)"
        fill="#ff9800"
        stroke="none"
        class="internal-tri-fill"
        :class="internalFillClass()"
      />

      <template v-for="(tri, ti) in internalTriangles" :key="`internal-edges-${ti}`">
        <line
          v-for="([p, q], ei) in triangleEdges(tri)"
          :key="`ite-${ti}-${ei}`"
          :x1="p.x"
          :y1="p.y"
          :x2="q.x"
          :y2="q.y"
          stroke="#ff9800"
          stroke-width="1.8"
          stroke-dasharray="5 4"
          stroke-linecap="round"
          class="internal-tri-edge"
          :class="{ 'is-dimmed': highlightCrossOnly }"
          :style="{ animationDelay: `${0.4 + ti * 0.12 + ei * 0.04}s` }"
        />
      </template>

      <g
        v-for="(e, i) in crossEdges"
        :key="`cross-${i}`"
        class="cross-edge-group"
        :class="{ 'is-visible': showCrossEdges }"
        :style="{
          transformOrigin: `${edgeMidpoint(e.x1, e.y1, e.x2, e.y2).x}px ${edgeMidpoint(e.x1, e.y1, e.x2, e.y2).y}px`,
          animationDelay: `${i * 0.07}s`,
        }"
      >
        <line
          :x1="e.x1"
          :y1="e.y1"
          :x2="e.x2"
          :y2="e.y2"
          stroke="#1976d2"
          stroke-width="2.5"
          stroke-linecap="round"
        />
      </g>

      <g
        v-for="(tri, ti) in crossTriangles"
        :key="`cross-group-${ti}`"
        class="cross-tri-group"
        :class="{ 'is-visible': showCrossEdges }"
        :style="{
          transformOrigin: `${triCentroid(tri).x}px ${triCentroid(tri).y}px`,
          animationDelay: `${0.12 + ti * 0.1}s`,
        }"
      >
        <polygon
          :points="pointsAttr(tri)"
          fill="#87ceeb"
          stroke="none"
          class="cross-tri-fill"
          :class="crossFillClass()"
        />

        <line
          v-for="([p, q], ei) in triangleEdges(tri)"
          :key="`cte-${ti}-${ei}`"
          :x1="p.x"
          :y1="p.y"
          :x2="q.x"
          :y2="q.y"
          :stroke="spansBoxes(p, q) ? '#1976d2' : '#ff9800'"
          :stroke-width="spansBoxes(p, q) ? 2.5 : 1.8"
          :stroke-dasharray="spansBoxes(p, q) ? undefined : '5 4'"
          stroke-linecap="round"
          class="cross-tri-edge"
          :class="{ 'is-dimmed': highlightCrossOnly && !spansBoxes(p, q) }"
        />

        <circle
          v-for="(p, pi) in [tri.a, tri.b, tri.c]"
          :key="`cv-${ti}-${pi}`"
          :cx="p.x"
          :cy="p.y"
          r="4"
          fill="#424242"
          class="tri-vertex cross-tri-vertex"
        />
      </g>

      <foreignObject
        v-for="(g, i) in graphs"
        :key="`label-${g.index}`"
        :x="g.x + boxW / 2 - labelBoxW / 2"
        :y="g.y + boxH + labelOffsetY - 6"
        :width="labelBoxW"
        :height="labelBoxH"
        class="graph-label-wrap"
        :style="{ animationDelay: `${0.15 + i * 0.1}s` }"
      >
        <div xmlns="http://www.w3.org/1999/xhtml" class="graph-label">
          <MathTex :tex="`G_{${g.index}}`" />
        </div>
      </foreignObject>

      <template v-for="(tri, ti) in internalTriangles" :key="`internal-verts-${ti}`">
        <circle
          v-for="(p, pi) in [tri.a, tri.b, tri.c]"
          :key="`iv-${ti}-${pi}`"
          :cx="p.x"
          :cy="p.y"
          r="4"
          fill="#424242"
          class="tri-vertex"
          :style="{ animationDelay: `${0.45 + ti * 0.12 + pi * 0.03}s` }"
        />
      </template>
    </svg>
  </div>
</template>

<style scoped>
.graph-embedding {
  width: 100%;
  max-width: 900px;
  margin: 0.5em auto 0;
}

svg {
  width: 100%;
  height: auto;
  display: block;
}

.graph-box {
  transform-origin: center;
  animation: graph-box-enter 0.55s ease-out both;
}

.graph-label-wrap {
  opacity: 0;
  animation: label-enter 0.4s ease-out both;
  overflow: visible;
}

.graph-label {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
  color: #e65100;
}

.graph-label :deep(.katex) {
  font-size: 1.05rem;
  color: #e65100;
}

.internal-tri-fill {
  opacity: 0;
  animation: internal-fill-enter 0.5s ease-out both;
}

.internal-tri-fill.is-default-fill {
  fill: #ff9800;
  fill-opacity: 0.22;
  opacity: 1;
}

.internal-tri-fill.is-no-fill {
  fill: #ff9800;
  fill-opacity: 0 !important;
  opacity: 1 !important;
  animation: none !important;
}

.internal-tri-fill.is-highlighted {
  fill: #ff9800 !important;
  opacity: 1 !important;
  animation: internal-orange-emphasis 0.85s ease-in-out infinite alternate !important;
}

.internal-tri-fill.is-dimmed {
  fill: #ff9800 !important;
  fill-opacity: 0.08 !important;
  opacity: 1 !important;
  animation: none !important;
}

.internal-tri-edge,
.tri-vertex:not(.cross-tri-vertex) {
  opacity: 0;
  animation: tri-enter 0.5s ease-out both;
}

.internal-tri-edge.is-dimmed {
  opacity: 0.25 !important;
}

.cross-edge-group,
.cross-tri-group {
  opacity: 0;
  transform: translateY(12px);
  pointer-events: none;
}

.cross-edge-group.is-visible,
.cross-tri-group.is-visible {
  animation: float-up-enter 0.52s ease-out both;
  pointer-events: auto;
}

.cross-tri-fill {
  fill: #87ceeb;
  fill-opacity: 0;
  transition: fill-opacity 0.35s ease;
}

.cross-tri-fill.is-no-fill {
  fill-opacity: 0 !important;
}

.cross-tri-fill.is-highlighted-sky {
  animation: cross-sky-blink 0.72s ease-in-out infinite alternate !important;
}

.cross-tri-edge.is-dimmed {
  opacity: 0.25 !important;
}

@keyframes internal-orange-emphasis {
  from {
    fill-opacity: 0.45;
  }

  to {
    fill-opacity: 0.88;
  }
}

@keyframes internal-fill-enter {
  from {
    opacity: 0;
  }

  to {
    opacity: 1;
  }
}

@keyframes cross-sky-blink {
  from {
    fill-opacity: 0.32;
  }

  to {
    fill-opacity: 0.78;
  }
}

@keyframes float-up-enter {
  from {
    opacity: 0;
    transform: translateY(12px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes graph-box-enter {
  from {
    opacity: 0;
    transform: scale(0.88);
  }

  to {
    opacity: 1;
    transform: scale(1);
  }
}

@keyframes label-enter {
  from {
    opacity: 0;
    transform: translateY(6px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes tri-enter {
  from {
    opacity: 0;
    transform: translateY(10px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>
