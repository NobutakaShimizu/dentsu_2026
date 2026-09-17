<script setup lang="ts">
import { computed, onUnmounted, ref, watch } from 'vue'
import { useNav } from '@slidev/client'
import MathTex from './Math.vue'

const N = 6
const D = 3
const L = 3
const AUTO_START_MS = 280
const AUTO_MIN_MS = 12
const AUTO_DECAY = 0.94
const LABELS = ['1', '2', '3', '4', '5', '6']
const COLORS = ['#1565c0', '#2e7d32', '#c2185b', '#ef6c00', '#6a1b9a', '#00838f']
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
  for (let seed = 3; seed < 400; seed++) {
    const g = tryRegular(mulberry32(seed))
    if (isRegular(g))
      return g.map(s => [...s].sort((a, b) => a - b))
  }
  return [
    [1, 2, 3],
    [0, 2, 4],
    [0, 1, 5],
    [0, 4, 5],
    [1, 3, 5],
    [2, 3, 4],
  ]
})()

function nbrs(v: number) {
  return adj[v]!
}

const walks = (() => {
  const forward: [number, number, number][] = []
  const back: [number, number, number][] = []
  for (let s = 0; s < N; s++) {
    for (const u of nbrs(s)) {
      for (const w of nbrs(u)) {
        const walk: [number, number, number] = [s, u, w]
        if (w === s)
          back.push(walk)
        else
          forward.push(walk)
      }
    }
  }
  return [...forward, ...back]
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

const A = Array.from({ length: N }, () => Array.from({ length: N }, () => 0))
const totalBlocks = walks.length * L

const { isPrintMode } = useNav()

const shown = ref(0)
const playing = ref(false)
let token = 0

function delay(ms: number, t: number) {
  return new Promise<void>((resolve) => {
    window.setTimeout(() => {
      if (t === token)
        resolve()
    }, ms)
  })
}

function stopAuto() {
  token += 1
  playing.value = false
}

function applyFinal() {
  stopAuto()
  shown.value = totalBlocks
}

function stepOnce() {
  if (isPrintMode.value)
    return
  stopAuto()
  shown.value = shown.value >= totalBlocks ? 1 : shown.value + 1
}

function stepBack() {
  if (isPrintMode.value)
    return
  stopAuto()
  shown.value = shown.value <= 0 ? totalBlocks : shown.value - 1
}

function autoDelay(step: number) {
  return Math.max(AUTO_MIN_MS, Math.round(AUTO_START_MS * AUTO_DECAY ** step))
}

async function playAuto() {
  if (isPrintMode.value)
    return
  const t = ++token
  playing.value = true
  if (shown.value >= totalBlocks)
    shown.value = 0
  let step = 0
  while (shown.value < totalBlocks) {
    if (t !== token)
      return
    shown.value += 1
    await delay(autoDelay(step), t)
    step += 1
  }
  if (t !== token)
    return
  playing.value = false
}

function toggleAuto() {
  if (playing.value)
    stopAuto()
  else
    playAuto()
}

watch(
  () => isPrintMode.value,
  (on) => {
    if (on)
      applyFinal()
  },
  { immediate: true },
)

onUnmounted(() => {
  stopAuto()
})

const done = computed(() => shown.value >= totalBlocks)

const activeWalk = computed(() => {
  if (shown.value <= 0 || done.value)
    return null
  return Math.floor((shown.value - 1) / L)
})

const activeBlock = computed(() => {
  if (activeWalk.value === null)
    return -1
  return (shown.value - 1) % L
})

const activeVerts = computed(() => {
  const wi = activeWalk.value
  if (wi === null)
    return [] as number[]
  return walks[wi]!
})

const nowVert = computed(() => {
  if (activeBlock.value < 0)
    return -1
  return activeVerts.value[activeBlock.value] ?? -1
})

function hexAlpha(hex: string, a: number) {
  const n = Number.parseInt(hex.slice(1), 16)
  const r = (n >> 16) & 255
  const g = (n >> 8) & 255
  const b = n & 255
  return `rgba(${r}, ${g}, ${b}, ${a})`
}

function aRowStyle(i: number) {
  const base = COLORS[i]!
  if (nowVert.value === i)
    return { background: hexAlpha(base, 0.55) }
  return { background: hexAlpha(base, 0.16) }
}

function blockVisible(wi: number, b: number) {
  return shown.value > wi * L + b
}

function blockNow(wi: number, b: number) {
  return activeWalk.value === wi && activeBlock.value === b
}

function rowNow(wi: number) {
  return activeWalk.value === wi
}

function edgeActive(a: number, b: number) {
  const verts = activeVerts.value
  if (verts.length < 2)
    return false
  for (let k = 0; k < verts.length - 1; k++) {
    const u = verts[k]!
    const v = verts[k + 1]!
    if ((u === a && v === b) || (u === b && v === a))
      return true
  }
  return false
}

function nodeNow(i: number) {
  return nowVert.value === i
}

function nodeStyle(i: number) {
  if (nodeNow(i))
    return { fill: COLORS[i] }
  return { fill: '#fff' }
}

const walkLabel = computed(() => {
  const verts = activeVerts.value
  if (!verts.length)
    return ''
  return `(${verts.map(v => LABELS[v]).join(', ')})`
})
</script>

<template>
  <div class="ewc">
    <div class="ewc-board">
      <div class="ewc-mat">
        <div class="ewc-name"><MathTex tex="A" /></div>
        <div class="ewc-agrid">
          <div
            v-for="(row, r) in A"
            :key="`A-${r}`"
            class="ewc-arow"
            :class="{ 'is-now': nowVert === r }"
          >
            <div class="ewc-row-idx">{{ LABELS[r] }}</div>
            <div
              v-for="(_val, c) in row"
              :key="`A-${r}-${c}`"
              class="ewc-acell"
              :style="aRowStyle(r)"
            ></div>
          </div>
        </div>
      </div>

      <div class="ewc-arrow" aria-hidden="true">
        <span class="ewc-arrow-line"></span>
      </div>

      <div class="ewc-mat ewc-ap">
        <div class="ewc-name"><MathTex tex="A'" /></div>
        <div class="ewc-ap-wrap">
          <div class="ewc-dim-v"><MathTex tex="|W|" /></div>
          <div class="ewc-apgrid">
            <div
              v-for="(walk, wi) in walks"
              :key="`w-${wi}`"
              class="ewc-aprow"
              :class="{ 'is-now': rowNow(wi) }"
            >
              <div
                v-for="(v, b) in walk"
                :key="`w-${wi}-b-${b}`"
                class="ewc-block"
                :class="{
                  'is-wait': !blockVisible(wi, b),
                  'is-on': blockVisible(wi, b),
                  'is-now': blockNow(wi, b),
                }"
              >
                <div
                  v-for="c in N"
                  :key="`w-${wi}-b-${b}-${c}`"
                  class="ewc-bcell"
                  :style="blockVisible(wi, b) ? { background: hexAlpha(COLORS[v]!, blockNow(wi, b) ? 0.7 : 0.45) } : undefined"
                ></div>
              </div>
            </div>
          </div>
        </div>
        <div class="ewc-dim-h"><MathTex tex="\ell n" /></div>
      </div>

      <div class="ewc-right">
        <svg
          class="ewc-graph"
          viewBox="0 0 220 216"
          role="img"
          aria-label="エクスパンダーグラフ"
        >
          <rect x="4" y="4" width="212" height="208" rx="18" class="ewc-graph-bg" />
          <line
            v-for="(e, i) in edges"
            :key="`e-${i}`"
            :x1="nodes[e[0]]!.x"
            :y1="nodes[e[0]]!.y"
            :x2="nodes[e[1]]!.x"
            :y2="nodes[e[1]]!.y"
            class="ewc-edge"
            :class="{ 'is-on': edgeActive(e[0], e[1]) }"
          />
          <g v-for="(p, i) in nodes" :key="`n-${i}`">
            <circle
              :cx="p.x"
              :cy="p.y"
              r="14"
              class="ewc-node"
              :class="{ 'is-now': nodeNow(i) }"
              :style="nodeStyle(i)"
            />
            <text
              :x="p.x"
              :y="p.y + 4"
              class="ewc-label"
              :class="{ 'is-now': nodeNow(i) }"
            >{{ p.label }}</text>
          </g>
        </svg>
        <div class="ewc-graph-meta">
          <div class="ewc-graph-cap">expander</div>
          <div class="ewc-walk" :class="{ 'is-on': !!walkLabel }">
            <MathTex tex="\mathbf{i}" />
            <span>{{ walkLabel || '\u00a0' }}</span>
          </div>
        </div>
      </div>
    </div>

    <div class="ewc-controls">
      <button
        type="button"
        class="ewc-btn"
        @click.stop="stepBack"
        @pointerdown.stop
      >
        一コマ戻す
      </button>
      <button
        type="button"
        class="ewc-btn"
        @click.stop="stepOnce"
        @pointerdown.stop
      >
        一コマ送り
      </button>
      <button
        type="button"
        class="ewc-btn"
        :class="{ 'is-on': playing }"
        @click.stop="toggleAuto"
        @pointerdown.stop
      >
        {{ playing ? '停止' : '自動コマ送り' }}
      </button>
    </div>
  </div>
</template>

<style scoped>
.ewc {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.55rem;
  margin: 0.15rem auto 0;
  width: fit-content;
  max-width: 100%;
}

.ewc-board {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.85rem;
}

.ewc-controls {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.45rem;
}

.ewc-btn {
  font-family: 'Roboto', sans-serif;
  font-size: 0.72rem;
  font-weight: 600;
  line-height: 1;
  color: #37474f;
  background: #fff;
  border: 1px solid #90a4ae;
  border-radius: 999px;
  padding: 0.38rem 0.85rem;
  cursor: pointer;
}

.ewc-btn:hover {
  border-color: #c2185b;
  color: #c2185b;
}

.ewc-btn.is-on {
  background: #f8bbd0;
  border-color: #c2185b;
  color: #c2185b;
}

.ewc-mat {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.18rem;
}

.ewc-name {
  font-weight: 700;
  font-size: 0.92rem;
  color: #37474f;
  line-height: 1.2rem;
  min-height: 1.2rem;
}

.ewc-agrid {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.ewc-arow {
  display: flex;
  align-items: center;
  gap: 0.22rem;
}

.ewc-row-idx {
  width: 0.7rem;
  text-align: right;
  font-family: 'Fira Code', ui-monospace, monospace;
  font-size: 0.62rem;
  font-weight: 700;
  color: #78909c;
}

.ewc-arow.is-now .ewc-row-idx {
  color: #c2185b;
}

.ewc-acell {
  width: 0.92rem;
  height: 0.92rem;
  border: 1px solid #90a4ae;
  box-sizing: border-box;
}

.ewc-arow.is-now .ewc-acell {
  border-color: #c2185b;
}

.ewc-arrow {
  display: flex;
  align-items: center;
  padding-top: 1.1rem;
}

.ewc-arrow-line {
  position: relative;
  display: block;
  width: 1.6rem;
  height: 0;
  border-top: 2px solid #546e7a;
}

.ewc-arrow-line::after {
  content: '';
  position: absolute;
  right: -1px;
  top: 50%;
  border-top: 5px solid transparent;
  border-bottom: 5px solid transparent;
  border-left: 8px solid #546e7a;
  transform: translateY(-50%);
}

.ewc-ap-wrap {
  display: flex;
  align-items: stretch;
  gap: 0.28rem;
}

.ewc-dim-v,
.ewc-dim-h {
  display: flex;
  align-items: center;
  justify-content: center;
  color: #607d8b;
  font-size: 0.72rem;
}

.ewc-dim-v {
  writing-mode: vertical-rl;
  transform: rotate(180deg);
}

.ewc-dim-h {
  margin-top: 0.12rem;
}

.ewc-apgrid {
  display: flex;
  flex-direction: column;
  gap: 1px;
  background: #cfd8dc;
  border: 1px solid #78909c;
  padding: 1px;
}

.ewc-aprow {
  display: flex;
  gap: 3px;
  background: #cfd8dc;
}

.ewc-aprow.is-now {
  outline: 1px solid #c2185b;
  outline-offset: 0;
  z-index: 1;
}

.ewc-block {
  display: flex;
  gap: 1px;
}

.ewc-bcell {
  width: 5px;
  height: 4px;
  background: #eceff1;
}

.ewc-block.is-wait .ewc-bcell {
  background: #eceff1;
}

.ewc-block.is-now .ewc-bcell {
  box-shadow: inset 0 0 0 1px #c2185b;
}

.ewc-right {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.12rem;
}

.ewc-graph {
  width: 12.4rem;
  height: auto;
  flex: 0 0 auto;
  overflow: visible;
}

.ewc-graph-bg {
  fill: #fafafa;
  stroke: #cfd8dc;
  stroke-width: 1;
}

.ewc-edge {
  stroke: #90a4ae;
  stroke-width: 1.15;
}

.ewc-edge.is-on {
  stroke: #c2185b;
  stroke-width: 2.4;
}

.ewc-node {
  fill: #fff;
  stroke: #546e7a;
  stroke-width: 1.4;
}

.ewc-label {
  fill: #37474f;
  font-family: 'Fira Code', ui-monospace, monospace;
  font-size: 11px;
  font-weight: 700;
  text-anchor: middle;
  pointer-events: none;
}

.ewc-label.is-now {
  fill: #fff;
}

.ewc-graph-meta {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.08rem;
  min-height: 2.1rem;
}

.ewc-graph-cap {
  font-family: 'Roboto', sans-serif;
  font-size: 0.68rem;
  font-weight: 600;
  color: #607d8b;
}

.ewc-walk {
  display: flex;
  align-items: baseline;
  gap: 0.18rem;
  min-height: 1.1rem;
  font-family: 'Fira Code', ui-monospace, monospace;
  font-size: 0.78rem;
  font-weight: 700;
  color: #c2185b;
  opacity: 0;
}

.ewc-walk.is-on {
  opacity: 1;
}
</style>
