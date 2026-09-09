<script setup lang="ts">
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

function edgesFromAdj(adj: Set<number>[]) {
  const edges: [number, number][] = []
  for (let i = 0; i < adj.length; i++) {
    for (const j of adj[i]!) {
      if (i < j)
        edges.push([i, j])
    }
  }
  return edges
}

function tryConfigRegular(n: number, d: number, rng: () => number) {
  const adj: Set<number>[] = Array.from({ length: n }, () => new Set())
  const stubs: number[] = []
  for (let i = 0; i < n; i++) {
    for (let k = 0; k < d; k++)
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
  return adj
}

function fillDegrees(adj: Set<number>[], d: number, rng: () => number, allow: (a: number, b: number) => boolean) {
  const n = adj.length
  for (let guard = 0; guard < n * d * 8; guard++) {
    const need: number[] = []
    for (let i = 0; i < n; i++) {
      for (let k = adj[i]!.size; k < d; k++)
        need.push(i)
    }
    if (need.length < 2)
      return
    shuffle(need, rng)
    let added = false
    for (let i = 0; i < need.length; i++) {
      for (let j = i + 1; j < need.length; j++) {
        const a = need[i]!
        const b = need[j]!
        if (a === b || adj[a]!.has(b) || !allow(a, b))
          continue
        if (adj[a]!.size >= d || adj[b]!.size >= d)
          continue
        adj[a]!.add(b)
        adj[b]!.add(a)
        added = true
        break
      }
      if (added)
        break
    }
    if (!added)
      return
  }
}

function randomRegular(n: number, d: number, seed: number) {
  const rng = mulberry32(seed)
  let best = tryConfigRegular(n, d, rng)
  let bestScore = edgesFromAdj(best).length
  for (let t = 0; t < 24; t++) {
    const adj = tryConfigRegular(n, d, rng)
    fillDegrees(adj, d, rng, () => true)
    const score = edgesFromAdj(adj).length
    if (score > bestScore) {
      best = adj
      bestScore = score
    }
    if (score >= (n * d) / 2)
      return edgesFromAdj(adj)
  }
  fillDegrees(best, d, rng, () => true)
  return edgesFromAdj(best)
}

function twoCommunities(n: number, d: number, bridges: number, seed: number) {
  const rng = mulberry32(seed)
  const half = n / 2
  const left = randomRegular(half, d, seed + 11)
  const right = randomRegular(half, d, seed + 29).map(([a, b]) => [a + half, b + half] as [number, number])
  const leftPick = shuffle(Array.from({ length: half }, (_, i) => i), rng).slice(0, bridges)
  const rightPick = shuffle(Array.from({ length: half }, (_, i) => i + half), rng).slice(0, bridges)
  const cross: [number, number][] = leftPick.map((a, i) => [a, rightPick[i]!])
  return { inner: [...left, ...right], cross }
}

function layoutForce(
  n: number,
  edges: [number, number][],
  seed: number,
  init: { x: number, y: number }[],
) {
  const rng = mulberry32(seed)
  const pos = init.map(p => ({
    x: p.x + (rng() - 0.5) * 0.02,
    y: p.y + (rng() - 0.5) * 0.02,
  }))
  const k = 1.15 / Math.sqrt(n)
  for (let iter = 0; iter < 90; iter++) {
    const cool = 0.18 * (1 - iter / 90)
    const disp = Array.from({ length: n }, () => ({ x: 0, y: 0 }))
    for (let i = 0; i < n; i++) {
      for (let j = i + 1; j < n; j++) {
        let dx = pos[i]!.x - pos[j]!.x
        let dy = pos[i]!.y - pos[j]!.y
        const dist = Math.hypot(dx, dy) || 0.001
        const f = (k * k) / dist
        dx = (dx / dist) * f
        dy = (dy / dist) * f
        disp[i]!.x += dx
        disp[i]!.y += dy
        disp[j]!.x -= dx
        disp[j]!.y -= dy
      }
    }
    for (const [a, b] of edges) {
      let dx = pos[a]!.x - pos[b]!.x
      let dy = pos[a]!.y - pos[b]!.y
      const dist = Math.hypot(dx, dy) || 0.001
      const f = (dist * dist) / k
      dx = (dx / dist) * f * 0.012
      dy = (dy / dist) * f * 0.012
      disp[a]!.x -= dx
      disp[a]!.y -= dy
      disp[b]!.x += dx
      disp[b]!.y += dy
    }
    for (let i = 0; i < n; i++) {
      const len = Math.hypot(disp[i]!.x, disp[i]!.y) || 1
      pos[i]!.x += (disp[i]!.x / len) * Math.min(len, cool)
      pos[i]!.y += (disp[i]!.y / len) * Math.min(len, cool)
    }
  }
  let minX = Infinity
  let minY = Infinity
  let maxX = -Infinity
  let maxY = -Infinity
  for (const p of pos) {
    minX = Math.min(minX, p.x)
    minY = Math.min(minY, p.y)
    maxX = Math.max(maxX, p.x)
    maxY = Math.max(maxY, p.y)
  }
  const sx = maxX - minX || 1
  const sy = maxY - minY || 1
  const pad = 3.2
  return pos.map(p => ({
    x: pad + ((p.x - minX) / sx) * (100 - pad * 2),
    y: pad + ((p.y - minY) / sy) * (100 - pad * 2),
  }))
}

function circleInit(n: number) {
  return Array.from({ length: n }, (_, i) => {
    const a = (2 * Math.PI * i) / n
    return { x: Math.cos(a), y: Math.sin(a) }
  })
}

function packCluster(pts: { x: number, y: number }[], x0: number, x1: number, y0: number, y1: number) {
  return pts.map(p => ({
    x: x0 + (p.x / 100) * (x1 - x0),
    y: y0 + (p.y / 100) * (y1 - y0),
  }))
}

const N = 100
const D = 6
const HALF = N / 2

const expanderEdges = randomRegular(N, D, 2026)
const { inner: sbmInner, cross: sbmCross } = twoCommunities(N, D, 3, 77)

const expanderPos = layoutForce(N, expanderEdges, 4, circleInit(N))

const leftEdges = sbmInner.filter(([a, b]) => a < HALF && b < HALF)
const rightEdges = sbmInner
  .filter(([a, b]) => a >= HALF && b >= HALF)
  .map(([a, b]) => [a - HALF, b - HALF] as [number, number])
const leftPos = packCluster(layoutForce(HALF, leftEdges, 9, circleInit(HALF)), 1.6, 44.8, 6, 94)
const rightPos = packCluster(layoutForce(HALF, rightEdges, 13, circleInit(HALF)), 55.2, 98.4, 6, 94)
const sbmPos = [...leftPos, ...rightPos]
</script>

<template>
  <div class="ex-cmp">
    <div class="ex-cmp-panel">
      <p class="ex-cmp-cap">エクスパンダー</p>
      <svg viewBox="0 0 100 100" class="ex-cmp-svg" role="img" aria-label="エクスパンダーグラフ">
        <line
          v-for="(e, i) in expanderEdges"
          :key="`e-${i}`"
          :x1="expanderPos[e[0]]!.x"
          :y1="expanderPos[e[0]]!.y"
          :x2="expanderPos[e[1]]!.x"
          :y2="expanderPos[e[1]]!.y"
          class="ex-cmp-edge"
        />
        <circle
          v-for="(p, i) in expanderPos"
          :key="`v-${i}`"
          :cx="p.x"
          :cy="p.y"
          r="0.55"
          class="ex-cmp-node"
        />
      </svg>
    </div>

    <div class="ex-cmp-rule" aria-hidden="true" />

    <div class="ex-cmp-panel">
      <p class="ex-cmp-cap">エクスパンダーじゃない</p>
      <svg viewBox="0 0 100 100" class="ex-cmp-svg" role="img" aria-label="非エクスパンダーグラフ">
        <line
          v-for="(e, i) in sbmInner"
          :key="`s-${i}`"
          :x1="sbmPos[e[0]]!.x"
          :y1="sbmPos[e[0]]!.y"
          :x2="sbmPos[e[1]]!.x"
          :y2="sbmPos[e[1]]!.y"
          class="ex-cmp-edge"
        />
        <line
          v-for="(e, i) in sbmCross"
          :key="`c-${i}`"
          :x1="sbmPos[e[0]]!.x"
          :y1="sbmPos[e[0]]!.y"
          :x2="sbmPos[e[1]]!.x"
          :y2="sbmPos[e[1]]!.y"
          class="ex-cmp-edge is-bridge"
        />
        <circle
          v-for="(p, i) in sbmPos"
          :key="`sv-${i}`"
          :cx="p.x"
          :cy="p.y"
          r="0.55"
          class="ex-cmp-node"
        />
      </svg>
    </div>
  </div>
</template>

<style scoped>
.ex-cmp {
  display: grid;
  grid-template-columns: 1fr 1px 1fr;
  gap: 0.85rem;
  align-items: stretch;
  height: 18.4rem;
  margin: 0.2rem 0 0;
}

.ex-cmp-rule {
  width: 1px;
  background: #90a4ae;
  align-self: stretch;
  margin: 0.15rem 0 0.35rem;
}

.ex-cmp-panel {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 0;
  min-width: 0;
  height: 100%;
}

.ex-cmp-cap {
  margin: 0 0 0.15rem;
  font-size: 0.95rem;
  font-weight: 700;
  color: #37474f;
  text-align: center;
  line-height: 1.2;
}

.ex-cmp-svg {
  display: block;
  width: 100%;
  flex: 1 1 auto;
  min-height: 0;
  overflow: visible;
}

.ex-cmp-edge {
  stroke: #546e7a;
  stroke-width: 0.22;
  stroke-opacity: 0.42;
}

.ex-cmp-edge.is-bridge {
  stroke: #78909c;
  stroke-width: 0.28;
  stroke-opacity: 0.7;
}

.ex-cmp-node {
  fill: #263238;
}
</style>
