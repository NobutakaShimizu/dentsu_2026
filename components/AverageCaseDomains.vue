<script setup lang="ts">
const panels = [
  {
    id: 'stats',
    label: 'Planted Clique',
    accent: '#d97706',
  },
  {
    id: 'crypto',
    label: 'Learning Parity with Noise',
    accent: '#1976d2',
  },
  {
    id: 'fgc',
    label: 'Triangle Counting',
    accent: '#7b1fa2',
  },
  {
    id: 'quantum',
    label: 'Quantum Circuit',
    accent: '#00838f',
  },
]

const lpnA = [
  [1, 0, 1],
  [0, 1, 1],
  [1, 1, 0],
]
const lpnS = [1, 0, 1]
const lpnE = [1, 0, 0]
const lpnB = [1, 1, 1]

const lpnCell = 10
const lpnGap = 2
const lpnAy = 30

function lpnBlockWidth(cols: number) {
  return cols * lpnCell + (cols - 1) * lpnGap
}

const lpnAx = 6
const aWidth = lpnBlockWidth(3)
const vWidth = lpnCell
const lpnMulX = lpnAx + aWidth + 5
const lpnSx = lpnMulX + 7
const lpnXorX = lpnSx + vWidth + 5
const lpnEx = lpnXorX + 7
const lpnEqX = lpnEx + vWidth + 5
const lpnBx = lpnEqX + 7
const lpnOpY = lpnAy + lpnCell + lpnGap + lpnCell / 2 + 1
const lpnLabelY = 22

function lpnCellPos(row: number, col: number) {
  return {
    x: lpnAx + col * (lpnCell + lpnGap),
    y: lpnAy + row * (lpnCell + lpnGap),
  }
}

function lpnColPos(row: number, colX: number) {
  return {
    x: colX,
    y: lpnAy + row * (lpnCell + lpnGap),
  }
}

function lpnArrow(fromX: number, toX: number) {
  return `M ${fromX} ${lpnOpY} L ${toX} ${lpnOpY}`
}

</script>

<template>
  <div class="avg-domains-grid" aria-label="Average-case complexity domains">
    <div
      v-for="(panel, i) in panels"
      :key="panel.id"
      class="avg-domain-panel"
      :class="`avg-domain-${panel.id}`"
      :style="{ '--panel-accent': panel.accent }"
    >
      <div class="avg-domain-orbit" aria-hidden="true" />
      <div class="avg-domain-glow" aria-hidden="true" />

      <!-- High-dimensional statistics: planted 5-clique -->
      <svg
        v-if="panel.id === 'stats'"
        class="avg-domain-svg"
        viewBox="0 0 140 96"
        xmlns="http://www.w3.org/2000/svg"
        role="img"
        aria-label="Planted clique"
      >
        <g class="avg-clique-bg">
          <circle cx="18" cy="22" r="3" fill="#cbd5e1" />
          <circle cx="32" cy="14" r="3" fill="#cbd5e1" />
          <circle cx="12" cy="42" r="3" fill="#cbd5e1" />
          <circle cx="28" cy="52" r="3" fill="#cbd5e1" />
          <circle cx="118" cy="18" r="3" fill="#cbd5e1" />
          <circle cx="128" cy="38" r="3" fill="#cbd5e1" />
          <circle cx="122" cy="62" r="3" fill="#cbd5e1" />
          <circle cx="108" cy="78" r="3" fill="#cbd5e1" />
          <circle cx="20" cy="78" r="3" fill="#cbd5e1" />
          <circle cx="124" cy="82" r="3" fill="#cbd5e1" />
          <line x1="18" y1="22" x2="32" y2="14" stroke="#e2e8f0" stroke-width="1" />
          <line x1="18" y1="22" x2="12" y2="42" stroke="#e2e8f0" stroke-width="1" />
          <line x1="32" y1="14" x2="28" y2="52" stroke="#e2e8f0" stroke-width="1" />
          <line x1="12" y1="42" x2="28" y2="52" stroke="#e2e8f0" stroke-width="1" />
          <line x1="118" y1="18" x2="128" y2="38" stroke="#e2e8f0" stroke-width="1" />
          <line x1="128" y1="38" x2="122" y2="62" stroke="#e2e8f0" stroke-width="1" />
          <line x1="122" y1="62" x2="108" y2="78" stroke="#e2e8f0" stroke-width="1" />
          <line x1="20" y1="78" x2="28" y2="52" stroke="#e2e8f0" stroke-width="1" />
          <line x1="108" y1="78" x2="124" y2="82" stroke="#e2e8f0" stroke-width="1" />
          <line x1="32" y1="14" x2="44" y2="38" stroke="#e2e8f0" stroke-width="1" opacity="0.6" />
          <line x1="96" y1="38" x2="118" y2="18" stroke="#e2e8f0" stroke-width="1" opacity="0.6" />
        </g>

        <g class="avg-clique-5">
          <line x1="70" y1="22" x2="96" y2="36" stroke="#ea580c" stroke-width="2" class="avg-draw" style="animation-delay: 0.5s" />
          <line x1="96" y1="36" x2="86" y2="64" stroke="#ea580c" stroke-width="2" class="avg-draw" style="animation-delay: 0.6s" />
          <line x1="86" y1="64" x2="54" y2="64" stroke="#ea580c" stroke-width="2" class="avg-draw" style="animation-delay: 0.7s" />
          <line x1="54" y1="64" x2="44" y2="36" stroke="#ea580c" stroke-width="2" class="avg-draw" style="animation-delay: 0.8s" />
          <line x1="44" y1="36" x2="70" y2="22" stroke="#ea580c" stroke-width="2" class="avg-draw" style="animation-delay: 0.9s" />
          <line x1="70" y1="22" x2="86" y2="64" stroke="#ea580c" stroke-width="1.5" class="avg-draw" style="animation-delay: 1.0s" />
          <line x1="70" y1="22" x2="54" y2="64" stroke="#ea580c" stroke-width="1.5" class="avg-draw" style="animation-delay: 1.1s" />
          <line x1="96" y1="36" x2="44" y2="36" stroke="#ea580c" stroke-width="1.5" class="avg-draw" style="animation-delay: 1.2s" />
          <line x1="96" y1="36" x2="54" y2="64" stroke="#ea580c" stroke-width="1.5" class="avg-draw" style="animation-delay: 1.3s" />
          <line x1="86" y1="64" x2="44" y2="36" stroke="#ea580c" stroke-width="1.5" class="avg-draw" style="animation-delay: 1.4s" />
          <ellipse cx="70" cy="44" rx="34" ry="28" fill="none" stroke="#f59e0b" stroke-width="1.5" stroke-dasharray="4 3" class="avg-clique-ring" />
          <circle cx="70" cy="22" r="5.5" fill="#f59e0b" stroke="#ea580c" stroke-width="1.5" class="avg-pulse" />
          <circle cx="96" cy="36" r="5.5" fill="#f59e0b" stroke="#ea580c" stroke-width="1.5" class="avg-pulse" style="animation-delay: 0.15s" />
          <circle cx="86" cy="64" r="5.5" fill="#f59e0b" stroke="#ea580c" stroke-width="1.5" class="avg-pulse" style="animation-delay: 0.3s" />
          <circle cx="54" cy="64" r="5.5" fill="#f59e0b" stroke="#ea580c" stroke-width="1.5" class="avg-pulse" style="animation-delay: 0.45s" />
          <circle cx="44" cy="36" r="5.5" fill="#f59e0b" stroke="#ea580c" stroke-width="1.5" class="avg-pulse" style="animation-delay: 0.6s" />
        </g>
      </svg>

      <!-- Cryptography: LPN instance generation -->
      <svg
        v-else-if="panel.id === 'crypto'"
        class="avg-domain-svg avg-lpn-svg"
        viewBox="0 0 140 96"
        xmlns="http://www.w3.org/2000/svg"
        role="img"
        aria-label="LPN instance generation"
      >
        <text class="avg-t-xs avg-lpn-formula" x="70" y="14" text-anchor="middle" fill="#64748b">b = As ⊕ e</text>

        <rect
          class="avg-lpn-scan"
          :x="lpnAx - 2"
          :y="lpnAy - 2"
          :width="aWidth + 4"
          :height="lpnCell + 4"
          rx="2"
        />

        <!-- 1. sample A -->
        <g class="avg-lpn-block avg-lpn-block-a">
          <text class="avg-t-xs avg-lpn-label" :x="lpnAx + aWidth / 2" :y="lpnLabelY" text-anchor="middle" fill="#1565c0">A</text>
          <template v-for="(row, ri) in lpnA" :key="`a-row-${ri}`">
            <g v-for="(bit, ci) in row" :key="`a-${ri}-${ci}`">
              <rect
                class="avg-lpn-cell"
                :x="lpnCellPos(ri, ci).x"
                :y="lpnCellPos(ri, ci).y"
                :width="lpnCell"
                :height="lpnCell"
                rx="2"
                :style="{ '--cell-delay': `${0.07 * (ri * 3 + ci)}s` }"
              />
              <text
                class="avg-mono avg-t-xs avg-lpn-bit"
                :x="lpnCellPos(ri, ci).x + lpnCell / 2"
                :y="lpnCellPos(ri, ci).y + lpnCell / 2 + 1.5"
                text-anchor="middle"
                dominant-baseline="middle"
                fill="#1565c0"
              >{{ bit }}</text>
            </g>
          </template>
        </g>

        <text class="avg-t-sm avg-lpn-op avg-lpn-op-mul" :x="lpnMulX" :y="lpnOpY" text-anchor="middle" fill="#64748b">·</text>

        <!-- 2. secret s -->
        <g class="avg-lpn-block avg-lpn-block-s">
          <text class="avg-t-xs avg-lpn-label" :x="lpnSx + vWidth / 2" :y="lpnLabelY" text-anchor="middle" fill="#1565c0">s</text>
          <g v-for="(bit, ri) in lpnS" :key="`s-${ri}`">
            <rect
              class="avg-lpn-cell avg-lpn-cell-secret"
              :x="lpnColPos(ri, lpnSx).x"
              :y="lpnColPos(ri, lpnSx).y"
              :width="lpnCell"
              :height="lpnCell"
              rx="2"
              :style="{ '--cell-delay': `${0.1 * ri}s` }"
            />
            <text
              class="avg-mono avg-t-xs avg-lpn-bit"
              :x="lpnSx + lpnCell / 2"
              :y="lpnColPos(ri, lpnSx).y + lpnCell / 2 + 1.5"
              text-anchor="middle"
              dominant-baseline="middle"
              fill="#1565c0"
            >{{ bit }}</text>
          </g>
        </g>

        <text class="avg-t-xs avg-lpn-op avg-lpn-op-xor" :x="lpnXorX" :y="lpnOpY" text-anchor="middle" fill="#c62828">⊕</text>

        <!-- 3. noise e -->
        <g class="avg-lpn-block avg-lpn-block-e">
          <text class="avg-t-xs avg-lpn-label" :x="lpnEx + vWidth / 2" :y="lpnLabelY" text-anchor="middle" fill="#c62828">e</text>
          <g v-for="(bit, ri) in lpnE" :key="`e-${ri}`">
            <rect
              class="avg-lpn-cell avg-lpn-cell-error"
              :x="lpnColPos(ri, lpnEx).x"
              :y="lpnColPos(ri, lpnEx).y"
              :width="lpnCell"
              :height="lpnCell"
              rx="2"
              :style="{ '--cell-delay': `${0.1 * ri}s` }"
            />
            <text
              class="avg-mono avg-t-xs avg-lpn-bit avg-lpn-bit-error"
              :x="lpnEx + lpnCell / 2"
              :y="lpnColPos(ri, lpnEx).y + lpnCell / 2 + 1.5"
              text-anchor="middle"
              dominant-baseline="middle"
              fill="#c62828"
            >{{ bit }}</text>
          </g>
        </g>

        <text class="avg-t-sm avg-lpn-op avg-lpn-op-eq" :x="lpnEqX" :y="lpnOpY" text-anchor="middle" fill="#64748b">=</text>

        <!-- 4. output b = As ⊕ e -->
        <g class="avg-lpn-block avg-lpn-block-b">
          <text class="avg-t-xs avg-lpn-label" :x="lpnBx + vWidth / 2" :y="lpnLabelY" text-anchor="middle" fill="#1565c0">b</text>
          <g v-for="(bit, ri) in lpnB" :key="`b-${ri}`">
            <rect
              class="avg-lpn-cell avg-lpn-cell-result"
              :x="lpnColPos(ri, lpnBx).x"
              :y="lpnColPos(ri, lpnBx).y"
              :width="lpnCell"
              :height="lpnCell"
              rx="2"
              :style="{ '--cell-delay': `${0.12 * ri}s` }"
            />
            <text
              class="avg-mono avg-t-xs avg-lpn-bit avg-lpn-bit-result"
              :x="lpnBx + lpnCell / 2"
              :y="lpnColPos(ri, lpnBx).y + lpnCell / 2 + 1.5"
              text-anchor="middle"
              dominant-baseline="middle"
              fill="#1565c0"
            >{{ bit }}</text>
          </g>
        </g>

        <!-- flow arrows -->
        <path class="avg-lpn-arrow avg-lpn-arrow-1" :d="lpnArrow(lpnAx + aWidth + 1, lpnSx - 1)" marker-end="url(#avg-lpn-arrowhead)" />
        <path class="avg-lpn-arrow avg-lpn-arrow-2" :d="lpnArrow(lpnSx + vWidth + 1, lpnEx - 1)" marker-end="url(#avg-lpn-arrowhead-red)" />
        <path class="avg-lpn-arrow avg-lpn-arrow-3" :d="lpnArrow(lpnEx + vWidth + 1, lpnBx - 1)" marker-end="url(#avg-lpn-arrowhead)" />

        <defs>
          <marker id="avg-lpn-arrowhead" markerWidth="5" markerHeight="5" refX="4" refY="2.5" orient="auto">
            <path d="M0,0 L5,2.5 L0,5 Z" fill="#94a3b8" />
          </marker>
          <marker id="avg-lpn-arrowhead-red" markerWidth="5" markerHeight="5" refX="4" refY="2.5" orient="auto">
            <path d="M0,0 L5,2.5 L0,5 Z" fill="#ef5350" />
          </marker>
        </defs>
      </svg>

      <TriangleCountingAnimation v-else-if="panel.id === 'fgc'" embedded />

      <!-- Quantum supremacy: random quantum circuit -->
      <svg
        v-else-if="panel.id === 'quantum'"
        class="avg-domain-svg avg-qc-svg"
        viewBox="0 0 140 96"
        xmlns="http://www.w3.org/2000/svg"
        role="img"
        aria-label="Random quantum circuit"
      >
        <!-- qubit wires -->
        <g class="avg-qc-wires">
          <line x1="12" y1="22" x2="128" y2="22" stroke="#00838f" stroke-width="1.5" opacity="0.35" />
          <line x1="12" y1="40" x2="128" y2="40" stroke="#00838f" stroke-width="1.5" opacity="0.35" />
          <line x1="12" y1="58" x2="128" y2="58" stroke="#00838f" stroke-width="1.5" opacity="0.35" />
          <line x1="12" y1="76" x2="128" y2="76" stroke="#00838f" stroke-width="1.5" opacity="0.35" />
          <text class="avg-t-xs" x="8" y="24" fill="#64748b">|0⟩</text>
          <text class="avg-t-xs" x="8" y="42" fill="#64748b">|0⟩</text>
          <text class="avg-t-xs" x="8" y="60" fill="#64748b">|0⟩</text>
          <text class="avg-t-xs" x="8" y="78" fill="#64748b">|0⟩</text>
        </g>

        <!-- layer 1: H gates -->
        <g class="avg-qc-layer avg-qc-layer-1">
          <rect x="22" y="14" width="16" height="16" rx="3" fill="rgba(0,131,143,0.15)" stroke="#00838f" stroke-width="1.5" class="avg-gate" />
          <text class="avg-t-sm" x="30" y="26" text-anchor="middle" fill="#006064">H</text>
          <rect x="22" y="32" width="16" height="16" rx="3" fill="rgba(0,131,143,0.15)" stroke="#00838f" stroke-width="1.5" class="avg-gate" style="animation-delay: 0.08s" />
          <text class="avg-t-sm" x="30" y="44" text-anchor="middle" fill="#006064">H</text>
          <rect x="22" y="50" width="16" height="16" rx="3" fill="rgba(0,131,143,0.15)" stroke="#00838f" stroke-width="1.5" class="avg-gate" style="animation-delay: 0.16s" />
          <text class="avg-t-sm" x="30" y="62" text-anchor="middle" fill="#006064">H</text>
          <rect x="22" y="68" width="16" height="16" rx="3" fill="rgba(0,131,143,0.15)" stroke="#00838f" stroke-width="1.5" class="avg-gate" style="animation-delay: 0.24s" />
          <text class="avg-t-sm" x="30" y="80" text-anchor="middle" fill="#006064">H</text>
        </g>

        <!-- layer 2: CNOT q0 → q1 -->
        <g class="avg-qc-layer avg-qc-layer-2">
          <circle cx="48" cy="22" r="3.5" fill="#00838f" class="avg-pulse" />
          <line x1="48" y1="22" x2="48" y2="36" stroke="#00838f" stroke-width="1.5" class="avg-draw" style="animation-delay: 0.1s" />
          <rect x="40" y="32" width="16" height="16" rx="3" fill="rgba(0,131,143,0.15)" stroke="#00838f" stroke-width="1.5" class="avg-gate" style="animation-delay: 0.15s" />
          <text class="avg-t-xs" x="48" y="44" text-anchor="middle" fill="#006064">⊕</text>
        </g>

        <!-- layer 3: T on q2, CNOT q2 → q3 -->
        <g class="avg-qc-layer avg-qc-layer-3">
          <rect x="62" y="50" width="16" height="16" rx="3" fill="rgba(0,131,143,0.15)" stroke="#00838f" stroke-width="1.5" class="avg-gate" />
          <text class="avg-t-sm" x="70" y="62" text-anchor="middle" fill="#006064">T</text>
          <circle cx="78" cy="58" r="3.5" fill="#00838f" class="avg-pulse" style="animation-delay: 0.1s" />
          <line x1="78" y1="58" x2="78" y2="72" stroke="#00838f" stroke-width="1.5" class="avg-draw" style="animation-delay: 0.2s" />
          <rect x="70" y="68" width="16" height="16" rx="3" fill="rgba(0,131,143,0.15)" stroke="#00838f" stroke-width="1.5" class="avg-gate" style="animation-delay: 0.25s" />
          <text class="avg-t-xs" x="78" y="80" text-anchor="middle" fill="#006064">⊕</text>
        </g>

        <!-- layer 4: H + CNOT q1 → q2 -->
        <g class="avg-qc-layer avg-qc-layer-4">
          <rect x="92" y="14" width="16" height="16" rx="3" fill="rgba(0,131,143,0.15)" stroke="#00838f" stroke-width="1.5" class="avg-gate" />
          <text class="avg-t-sm" x="100" y="26" text-anchor="middle" fill="#006064">H</text>
          <circle cx="92" cy="40" r="3.5" fill="#00838f" class="avg-pulse" style="animation-delay: 0.05s" />
          <line x1="92" y1="40" x2="92" y2="54" stroke="#00838f" stroke-width="1.5" class="avg-draw" style="animation-delay: 0.15s" />
          <rect x="84" y="50" width="16" height="16" rx="3" fill="rgba(0,131,143,0.15)" stroke="#00838f" stroke-width="1.5" class="avg-gate" style="animation-delay: 0.2s" />
          <text class="avg-t-xs" x="92" y="62" text-anchor="middle" fill="#006064">⊕</text>
        </g>

        <!-- measurement -->
        <g class="avg-qc-layer avg-qc-layer-5">
          <line x1="118" y1="14" x2="118" y2="82" stroke="#00838f" stroke-width="2" class="avg-qc-measure" />
          <line x1="124" y1="14" x2="124" y2="82" stroke="#00838f" stroke-width="2" class="avg-qc-measure" style="animation-delay: 0.08s" />
        </g>

        <!-- signal pulse traveling along wires -->
        <circle cx="12" cy="22" r="2.5" fill="#00838f" class="avg-qc-signal avg-qc-signal-0" />
        <circle cx="12" cy="40" r="2.5" fill="#00838f" class="avg-qc-signal avg-qc-signal-1" />
        <circle cx="12" cy="58" r="2.5" fill="#00838f" class="avg-qc-signal avg-qc-signal-2" />
      </svg>

      <span class="avg-domain-label">{{ panel.label }}</span>
    </div>
  </div>
</template>
