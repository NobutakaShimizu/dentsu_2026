<script setup lang="ts">
import { computed, onUnmounted, ref, watch } from 'vue'
import { useNav, useSlideContext } from '@slidev/client'
import MathTex from './Math.vue'

const N = 11
const D = 4
const LABELS = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k']
const X = [1, 0, 1, 1, 1, 0, 0, 1, 0, 0, 1]
const COLS = 16
const CX = 110
const CY = 108
const R = 82

function mulberry32(seed: number) {
  let s = seed >>> 0
  return () => {
    s = (s + 0x6D2B79F5) >>> 0
    let t = Math.imul(s ^ (s >>> 15), 1 | s)
    t = (t + Math.imul(t ^ (t >>> 7), 61 | t)) ^ t
    return ((t ^ (t >>> 14)) >>> 0) / 4294967296
  }
}

function shuffle<T>(arr: T[], rng: () => number) {
  for (let i = arr.length - 1; i > 0; i--) {
    const j = Math.floor(rng() * (i + 1))
    const tmp = arr[i]!
    arr[i] = arr[j]!
    arr[j] = tmp
  }
  return arr
}

function tryRegular(rng: () => number) {
  const adj: Set<number>[] = Array.from({ length: N }, () => new Set())
  const stubs: number[] = []
  for (let i = 0; i < N; i++) {
    for (let k = 0; k < D; k++)
      stubs.push(i)
  }
  shuffle(stubs, rng)
  for (let i = 0; i + 1 < stubs.length; i += 2) {
    const a = stubs[i]!
    const b = stubs[i + 1]!
    if (a === b || adj[a]!.has(b))
      continue
    adj[a]!.add(b)
    adj[b]!.add(a)
  }
  for (let guard = 0; guard < N * D * 12; guard++) {
    const need: number[] = []
    for (let i = 0; i < N; i++) {
      for (let k = adj[i]!.size; k < D; k++)
        need.push(i)
    }
    if (need.length < 2)
      break
    shuffle(need, rng)
    let added = false
    for (let i = 0; i < need.length && !added; i++) {
      for (let j = i + 1; j < need.length; j++) {
        const a = need[i]!
        const b = need[j]!
        if (a === b || adj[a]!.has(b) || adj[a]!.size >= D || adj[b]!.size >= D)
          continue
        adj[a]!.add(b)
        adj[b]!.add(a)
        added = true
        break
      }
    }
    if (!added)
      break
  }
  return adj
}

function isRegular(adj: Set<number>[]) {
  return adj.every(s => s.size === D)
}

const adj = (() => {
  for (let seed = 11; seed < 400; seed++) {
    const g = tryRegular(mulberry32(seed))
    if (isRegular(g))
      return g.map(s => [...s].sort((a, b) => a - b))
  }
  return Array.from({ length: N }, (_, i) =>
    [1, 2, N - 1, N - 2].map(d => (i + d) % N).sort((a, b) => a - b),
  )
})()

function nbrs(v: number) {
  return adj[v]!
}

const walks = (() => {
  const list: { verts: [number, number, number], val: number }[] = []
  for (let s = 0; s < N; s++) {
    for (const u of nbrs(s)) {
      for (const w of nbrs(u)) {
        list.push({
          verts: [s, u, w],
          val: (X[s]! + X[u]! + X[w]!) % 2,
        })
      }
    }
  }
  return list
})()

const edges = (() => {
  const seen = new Set<string>()
  const list: [number, number][] = []
  for (let v = 0; v < N; v++) {
    for (const u of nbrs(v)) {
      const a = Math.min(v, u)
      const b = Math.max(v, u)
      const key = `${a}-${b}`
      if (seen.has(key))
        continue
      seen.add(key)
      list.push([a, b])
    }
  }
  return list
})()

const nodes = Array.from({ length: N }, (_, i) => {
  const ang = -Math.PI / 2 + (2 * Math.PI * i) / N
  return {
    x: CX + R * Math.cos(ang),
    y: CY + R * Math.sin(ang),
    label: LABELS[i]!,
  }
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
  shown.value = walks.length
}

async function playFill(t: number) {
  shown.value = 0
  await delay(160, t)
  for (let k = 1; k <= walks.length; k++) {
    if (t !== token)
      return
    shown.value = k
    await delay(320, t)
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

const done = computed(() => shown.value >= walks.length)

const active = computed(() => {
  if (shown.value <= 0 || shown.value > walks.length || done.value)
    return null
  return walks[shown.value - 1]!
})

const activeSet = computed(() => {
  const a = active.value
  if (!a)
    return new Set<number>()
  return new Set(a.verts)
})

function edgeActive(a: number, b: number) {
  const w = active.value
  if (!w)
    return false
  const [u, v, t] = w.verts
  return (u === a && v === b) || (u === b && v === a)
    || (v === a && t === b) || (v === b && t === a)
}

function xActive(k: number) {
  return activeSet.value.has(k)
}

function encVisible(t: number) {
  return t < shown.value
}

function encActive(t: number) {
  return !done.value && t === shown.value - 1
}
</script>

<template>
  <div class="ew">
    <div v-click class="ew-click" aria-hidden="true" />

    <div class="ew-left">
      <div class="ew-row">
        <div class="ew-name"><MathTex tex="x" /></div>
        <div class="ew-vec">
          <div
            v-for="(v, k) in X"
            :key="`x-${k}`"
            class="ew-cell"
            :class="{ 'is-src': xActive(k) }"
          >{{ v }}</div>
        </div>
      </div>

      <div class="ew-row ew-arrow-row">
        <div class="ew-name" aria-hidden="true" />
        <div class="ew-arrow">
          <span class="ew-arrow-line" />
        </div>
      </div>

      <div class="ew-row ew-enc-row">
        <div class="ew-name"><MathTex tex="\mathrm{Enc}(x)" /></div>
        <div class="ew-grid">
          <div
            v-for="(w, t) in walks"
            :key="`e-${t}`"
            class="ew-cell is-sm"
            :class="{
              'is-wait': !encVisible(t),
              'is-on': encVisible(t),
              'is-now': encActive(t),
            }"
          >{{ encVisible(t) ? w.val : '' }}</div>
        </div>
      </div>
    </div>

    <svg
      class="ew-graph"
      viewBox="0 0 220 216"
      role="img"
      aria-label="11頂点エクスパンダーグラフ"
    >
      <rect
        x="4"
        y="4"
        width="212"
        height="208"
        rx="18"
        class="ew-graph-bg"
      />
      <line
        v-for="(e, i) in edges"
        :key="`e-${i}`"
        :x1="nodes[e[0]]!.x"
        :y1="nodes[e[0]]!.y"
        :x2="nodes[e[1]]!.x"
        :y2="nodes[e[1]]!.y"
        class="ew-edge"
        :class="{ 'is-on': edgeActive(e[0], e[1]) }"
      />
      <g v-for="(p, i) in nodes" :key="`n-${i}`">
        <circle
          :cx="p.x"
          :cy="p.y"
          r="11.5"
          class="ew-node"
          :class="{ 'is-on': xActive(i) }"
        />
        <text
          :x="p.x"
          :y="p.y + 3.6"
          class="ew-label"
          :class="{ 'is-on': xActive(i) }"
        >{{ p.label }}</text>
      </g>
    </svg>
  </div>
</template>

<style scoped>
.ew {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1.35rem;
  margin: 0.45rem auto 0;
  width: fit-content;
  max-width: 100%;
}

.ew-click {
  position: absolute;
  width: 0;
  height: 0;
  overflow: hidden;
}

.ew-left {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 0.16rem;
}

.ew-row {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.ew-enc-row {
  align-items: flex-start;
  padding-top: 0.15rem;
}

.ew-name {
  flex: 0 0 4.8rem;
  text-align: right;
  font-size: 0.9rem;
  font-weight: 700;
  color: #37474f;
  line-height: 1.45rem;
}

.ew-vec {
  display: flex;
  gap: 1px;
  background: #b0bec5;
  border: 1px solid #78909c;
  padding: 1px;
}

.ew-grid {
  display: grid;
  grid-template-columns: repeat(16, 0.78rem);
  gap: 1px;
  background: #b0bec5;
  border: 1px solid #78909c;
  padding: 1px;
}

.ew-cell {
  width: 1.22rem;
  height: 1.22rem;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #fff;
  color: #455a64;
  font-family: 'Fira Code', monospace;
  font-size: 0.7rem;
  line-height: 1;
}

.ew-cell.is-sm {
  width: 0.78rem;
  height: 0.78rem;
  font-size: 0.52rem;
}

.ew-cell.is-src,
.ew-cell.is-now {
  background: #f8bbd0;
  color: #c2185b;
  font-weight: 700;
}

.ew-cell.is-wait {
  background: #eceff1;
  color: transparent;
}

.ew-cell.is-on {
  background: #fff;
  color: #37474f;
}

.ew-arrow-row {
  align-items: stretch;
}

.ew-arrow {
  display: flex;
  justify-content: center;
  width: calc(11 * 1.22rem + 13px);
}

.ew-arrow-line {
  position: relative;
  display: block;
  width: 0;
  height: 2.2rem;
  border-left: 2px solid #546e7a;
}

.ew-arrow-line::after {
  content: '';
  position: absolute;
  left: -6px;
  bottom: -2px;
  border-left: 5px solid transparent;
  border-right: 5px solid transparent;
  border-top: 9px solid #546e7a;
}

.ew-graph {
  width: 13.4rem;
  height: auto;
  flex: 0 0 auto;
  overflow: visible;
}

.ew-graph-bg {
  fill: #fafafa;
  stroke: #cfd8dc;
  stroke-width: 1;
}

.ew-edge {
  stroke: #90a4ae;
  stroke-width: 1.15;
}

.ew-edge.is-on {
  stroke: #c2185b;
  stroke-width: 2.35;
}

.ew-node {
  fill: #fff;
  stroke: #546e7a;
  stroke-width: 1.15;
}

.ew-node.is-on {
  fill: #f8bbd0;
  stroke: #c2185b;
}

.ew-label {
  fill: #37474f;
  font-family: 'Fira Code', ui-monospace, monospace;
  font-size: 11px;
  font-weight: 700;
  text-anchor: middle;
  pointer-events: none;
}

.ew-label.is-on {
  fill: #c2185b;
}
</style>
