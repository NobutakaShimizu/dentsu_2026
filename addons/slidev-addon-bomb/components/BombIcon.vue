<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{
  heat: number
  exploded: boolean
  ignited: boolean
  current: number
  total: number
  showSlideNum: boolean
}>()

const heatClamped = computed(() => Math.min(1, Math.max(0, props.heat)))
const isCritical = computed(() => !props.exploded && (props.ignited || heatClamped.value >= 0.75))
const glowStrength = computed(() => {
  if (props.ignited)
    return 1
  return heatClamped.value * 0.55
})

const sparkAngles = Array.from({ length: 48 }, (_, i) => i * (360 / 48))
const trailAngles = Array.from({ length: 32 }, (_, i) => i * (360 / 32) + 6)
const fireworkBursts = [
  { delay: '0s', x: '0px', y: '0px' },
  { delay: '0.08s', x: '-42px', y: '-48px' },
  { delay: '0.14s', x: '46px', y: '-36px' },
  { delay: '0.2s', x: '-28px', y: '32px' },
  { delay: '0.26s', x: '36px', y: '28px' },
  { delay: '0.32s', x: '-52px', y: '8px' },
  { delay: '0.38s', x: '18px', y: '-52px' },
]
</script>

<template>
  <div
    class="bomb-unit suika-unit"
    :class="{
      'is-critical': isCritical,
      'is-ignited': ignited && !exploded,
      'is-exploded': exploded,
    }"
    :style="{ '--heat-glow': glowStrength }"
  >
    <div v-if="exploded" class="bomb-explosion" aria-hidden="true">
      <span class="fw-flash" />
      <span class="fw-ring fw-ring-1" />
      <span class="fw-ring fw-ring-2" />
      <span class="fw-ring fw-ring-3" />
      <span
        v-for="(angle, i) in sparkAngles"
        :key="`spark-${i}`"
        class="fw-spark"
        :class="`fw-spark-${i % 8}`"
        :style="{
          '--angle': `${angle}deg`,
          '--delay': `${(i % 9) * 0.03}s`,
          '--dist': `${58 + (i % 6) * 16}px`,
        }"
      />
      <span
        v-for="(angle, i) in trailAngles"
        :key="`trail-${i}`"
        class="fw-trail"
        :style="{
          '--angle': `${angle}deg`,
          '--delay': `${0.04 + (i % 6) * 0.025}s`,
          '--dist': `${90 + (i % 5) * 24}px`,
          '--trail-color': ['#ff1744', '#ff9100', '#ffea00', '#76ff03', '#2979ff', '#e040fb', '#ff4081', '#00e5ff'][i % 8],
        }"
      />
      <span
        v-for="(burst, i) in fireworkBursts"
        :key="`burst-${i}`"
        class="fw-secondary"
        :style="{
          '--delay': burst.delay,
          '--bx': burst.x,
          '--by': burst.y,
        }"
      />
    </div>

    <template v-else>
      <div class="suika-glow" aria-hidden="true" />

      <svg
        viewBox="0 0 80 56"
        class="bomb-svg suika-svg"
        aria-hidden="true"
      >
        <defs>
          <clipPath id="suika-clip">
            <ellipse cx="42" cy="30" rx="30" ry="24" />
          </clipPath>
        </defs>

        <!-- rind -->
        <ellipse cx="42" cy="30" rx="30" ry="24" class="suika-rind" />

        <!-- dark stripes -->
        <g clip-path="url(#suika-clip)">
          <path d="M18 6 Q28 30 18 54" class="suika-stripe" />
          <path d="M30 6 Q38 30 30 54" class="suika-stripe" />
          <path d="M42 6 Q42 30 42 54" class="suika-stripe" />
          <path d="M54 6 Q46 30 54 54" class="suika-stripe" />
          <path d="M66 6 Q56 30 66 54" class="suika-stripe" />
        </g>

        <!-- highlight -->
        <ellipse cx="30" cy="22" rx="8" ry="5" class="suika-shine" />
        <!-- stem + leaf -->
        <path d="M42 8 C41 4 43 2 44 1" class="suika-stem" />
        <path d="M44 3 C48 1 52 4 50 8 C48 6 46 5 44 3 Z" class="suika-leaf" />

        <text
          v-if="showSlideNum"
          x="42"
          y="34"
          text-anchor="middle"
          class="bomb-page-num suika-page-num"
        >
          {{ current }}/{{ total }}
        </text>
      </svg>
    </template>
  </div>
</template>
