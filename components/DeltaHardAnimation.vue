<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'

const inputs = ['000', '001', '010', '011', '100', '101', '110', '111'] as const
const fTable = [0, 1, 1, 0, 1, 0, 0, 1]

const INPUT_X = 2
const INPUT_YS = [14, 26, 38] as const
const OUTLINE = '14,6 14,46 108,26'

type Wire = { x1: number, y1: number, x2: number, y2: number }
type Gate = { cx: number, cy: number, r: number }
type Dot = { cx: number, cy: number }

type Signal =
  | { kind: 'in', idx: number }
  | { kind: 'gate', id: number }

type CircuitArt = {
  outline: string
  wires: Wire[]
  gates: Gate[]
  dots: Dot[]
  outputs: number[]
  numGates: number
}

type Phase = 'circuit' | 'table' | 'compare'

const PHASE_MS: Record<Phase, number> = {
  circuit: 2000,
  table: 2400,
  compare: 3600,
}

function mulberry32(seed: number) {
  let s = seed >>> 0
  return () => {
    s = (s + 0x6D2B79F5) >>> 0
    let t = Math.imul(s ^ (s >>> 15), 1 | s)
    t = (t + Math.imul(t ^ (t >>> 7), 61 | t)) ^ t
    return ((t ^ (t >>> 14)) >>> 0) / 4294967296
  }
}

function signalLayer(sig: Signal, gateLayers: number[]): number {
  if (sig.kind === 'in') return 0
  return gateLayers[sig.id]
}

function signalAnchor(sig: Signal, gatePos: { cx: number, cy: number }[]): { x: number, y: number } {
  if (sig.kind === 'in') return { x: 14, y: INPUT_YS[sig.idx] }
  const g = gatePos[sig.id]
  return { x: g.cx + 4.5, y: g.cy }
}

function gatePort(gate: Gate, port: 0 | 1): { x: number, y: number } {
  const offset = port === 0 ? -2.2 : 2.2
  return { x: gate.cx - gate.r * 0.72, y: gate.cy + offset }
}

function routeWire(from: { x: number, y: number }, to: { x: number, y: number }, bendX: number): Wire[] {
  return [
    { x1: from.x, y1: from.y, x2: bendX, y2: from.y },
    { x1: bendX, y1: from.y, x2: bendX, y2: to.y },
    { x1: bendX, y1: to.y, x2: to.x, y2: to.y },
  ]
}

function genOutputs(rng: () => number): number[] {
  const out = [...fTable]
  const errors = 2 + Math.floor(rng() * 3)
  const idxs = new Set<number>()
  while (idxs.size < errors) idxs.add(Math.floor(rng() * 8))
  for (const i of idxs) out[i] = 1 - out[i]
  return out
}

function buildCircuit(seed: number): CircuitArt {
  const rng = mulberry32(seed)
  const numGates = 3 + Math.floor(rng() * 3)

  const gateInputs: { a: Signal, b: Signal }[] = []
  const gateLayers: number[] = []
  let signals: Signal[] = [
    { kind: 'in', idx: 0 },
    { kind: 'in', idx: 1 },
    { kind: 'in', idx: 2 },
  ]

  for (let i = 0; i < numGates; i++) {
    const ai = Math.floor(rng() * signals.length)
    let bi = Math.floor(rng() * signals.length)
    if (bi === ai && signals.length > 1) bi = (bi + 1 + Math.floor(rng() * (signals.length - 1))) % signals.length

    const a = signals[ai]
    const b = signals[bi]
    const layer = Math.max(signalLayer(a, gateLayers), signalLayer(b, gateLayers)) + 1

    gateInputs.push({ a, b })
    gateLayers.push(layer)
    signals.push({ kind: 'gate', id: i })
  }

  const maxLayer = Math.max(...gateLayers)
  const layerCounts = Array.from({ length: maxLayer + 1 }, () => 0)
  for (const l of gateLayers) layerCounts[l]++

  const layerSlots = layerCounts.map(c => 0)
  const gatePos: { cx: number, cy: number }[] = []

  for (let i = 0; i < numGates; i++) {
    const layer = gateLayers[i]
    layerSlots[layer]++
    const count = layerCounts[layer]
    const slot = layerSlots[layer]
    const cx = maxLayer === 0 ? 52 : 24 + (layer / maxLayer) * 58
    const cy = 8 + (slot / (count + 1)) * 36
    gatePos.push({ cx, cy })
  }

  const gates: Gate[] = gatePos.map((p, i) => ({
    cx: p.cx,
    cy: p.cy,
    r: 4.2 + (i % 3) * 0.35,
  }))

  const wires: Wire[] = []
  const dots: Dot[] = []

  for (let i = 0; i < 3; i++) {
    wires.push({ x1: INPUT_X, y1: INPUT_YS[i], x2: 14, y2: INPUT_YS[i] })
  }

  for (let i = 0; i < numGates; i++) {
    const gate = gates[i]
    const { a, b } = gateInputs[i]
    const bendA = gate.cx - 10 - (i % 2) * 3
    const bendB = gate.cx - 6 - (i % 3) * 2

    const fromA = signalAnchor(a, gatePos)
    const fromB = signalAnchor(b, gatePos)
    const toA = gatePort(gate, 0)
    const toB = gatePort(gate, 1)

    for (const [from, to, bend] of [[fromA, toA, bendA], [fromB, toB, bendB]] as const) {
      const segs = routeWire(from, to, bend)
      wires.push(...segs)
      dots.push({ cx: bend, cy: from.y })
      dots.push({ cx: bend, cy: to.y })
    }
  }

  const last = gates[numGates - 1]
  wires.push({ x1: last.cx + last.r * 0.75, y1: last.cy, x2: 108, y2: last.cy })

  return {
    outline: OUTLINE,
    wires,
    gates,
    dots,
    outputs: genOutputs(rng),
    numGates,
  }
}

function triPoints(cx: number, cy: number, r: number) {
  const h = r * 0.82
  const nose = r * 0.55
  const tail = r * 0.42
  return `${cx + nose},${cy} ${cx - tail},${cy - h} ${cx - tail},${cy + h}`
}

const circuitIndex = ref(0)
const phase = ref<Phase>('circuit')
const activeArt = ref<CircuitArt>(buildCircuit(1))

let timer: ReturnType<typeof setTimeout> | undefined

function scheduleNext() {
  timer = setTimeout(tick, PHASE_MS[phase.value])
}

function tick() {
  if (phase.value === 'circuit') phase.value = 'table'
  else if (phase.value === 'table') phase.value = 'compare'
  else {
    phase.value = 'circuit'
    circuitIndex.value += 1
    activeArt.value = buildCircuit(circuitIndex.value + 1)
  }
  scheduleNext()
}

onMounted(() => {
  activeArt.value = buildCircuit(1)
  scheduleNext()
})

onUnmounted(() => {
  if (timer) clearTimeout(timer)
})
</script>

<template>
  <div
    class="dh-anim"
    :class="`dh-phase-${phase}`"
    aria-label="Animation: comparing truth tables of f and small circuits"
  >
    <div class="dh-table">
      <div class="dh-row dh-head">
        <span class="dh-label">x</span>
        <span v-for="input in inputs" :key="`h-${input}`" class="dh-cell dh-head-cell">{{ input }}</span>
      </div>

      <div class="dh-row dh-row-f">
        <span class="dh-label">f(x)</span>
        <span v-for="(value, index) in fTable" :key="`f-${index}`" class="dh-cell dh-cell-f">{{ value }}</span>
      </div>

      <div :key="`${circuitIndex}-${activeArt.numGates}`" class="dh-circuit-stage">
        <div class="dh-circuit-meta">
          <svg
            class="dh-circuit-svg"
            viewBox="0 0 112 52"
            xmlns="http://www.w3.org/2000/svg"
            role="img"
            :aria-label="`Circuit C sub ${circuitIndex + 1}`"
          >
            <polygon :points="activeArt.outline" class="dh-tri-outline" />
            <line
              v-for="(wire, wi) in activeArt.wires"
              :key="`w-${circuitIndex}-${wi}`"
              :x1="wire.x1"
              :y1="wire.y1"
              :x2="wire.x2"
              :y2="wire.y2"
              class="dh-wire"
            />
            <circle
              v-for="(dot, di) in activeArt.dots"
              :key="`d-${circuitIndex}-${di}`"
              :cx="dot.cx"
              :cy="dot.cy"
              r="1.1"
              class="dh-junction"
            />
            <polygon
              v-for="(gate, gi) in activeArt.gates"
              :key="`g-${circuitIndex}-${gi}`"
              :points="triPoints(gate.cx, gate.cy, gate.r)"
              class="dh-tri-gate"
            />
          </svg>
          <span class="dh-circuit-name">C<sub>{{ circuitIndex + 1 }}</sub></span>
        </div>

        <div class="dh-row dh-row-c">
          <span class="dh-label">C(x)</span>
          <span
            v-for="(value, index) in activeArt.outputs"
            :key="`c-${circuitIndex}-${index}`"
            class="dh-cell dh-cell-c"
            :class="{
              'dh-cell-show': phase !== 'circuit',
              'dh-mismatch': phase === 'compare' && value !== fTable[index],
            }"
            :style="{ transitionDelay: `${index * 0.05}s` }"
          >{{ phase === 'circuit' ? '·' : value }}</span>
        </div>
      </div>
    </div>
  </div>
</template>
