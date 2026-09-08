<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref } from 'vue'
import {
  fgcEdgeKey,
  fgcEdges,
  fgcNodes,
  fgcTriEdges,
  fgcTriPoints,
  fgcTriangles,
} from './triangleCountingData'

withDefaults(defineProps<{
  embedded?: boolean
}>(), {
  embedded: false,
})

const TRI_ENUM_MS = 130

const activeIndex = ref(0)
const activeTri = computed(() => fgcTriangles[activeIndex.value])

let timer: ReturnType<typeof setInterval> | undefined

onMounted(() => {
  timer = setInterval(() => {
    activeIndex.value = (activeIndex.value + 1) % fgcTriangles.length
  }, TRI_ENUM_MS)
})

onUnmounted(() => {
  if (timer)
    clearInterval(timer)
})
</script>

<template>
  <div
    class="tri-count-anim"
    :class="{ 'tri-count-anim-embedded': embedded }"
    aria-label="Triangle counting animation"
  >
    <svg
      class="avg-domain-svg avg-fgc-svg"
      viewBox="4 4 74 64"
      xmlns="http://www.w3.org/2000/svg"
      role="img"
      aria-label="Subgraph counting: enumerate triangles quickly"
    >
      <g class="avg-fgc-graph">
        <line
          v-for="(edge, ei) in fgcEdges"
          :key="`e-${fgcEdgeKey(edge[0], edge[1])}`"
          :x1="fgcNodes[edge[0]].x"
          :y1="fgcNodes[edge[0]].y"
          :x2="fgcNodes[edge[1]].x"
          :y2="fgcNodes[edge[1]].y"
          stroke="#cbd5e1"
          stroke-width="0.8"
          class="avg-fgc-edge"
        />
        <circle
          v-for="(node, ni) in fgcNodes"
          :key="`n-${ni}`"
          :cx="node.x"
          :cy="node.y"
          r="2"
          fill="#94a3b8"
          stroke="#64748b"
          stroke-width="0.5"
          class="avg-fgc-node"
        />
      </g>

      <g
        v-if="activeTri"
        :key="activeIndex"
        class="avg-fgc-tri avg-fgc-tri-active"
      >
        <polygon
          :points="fgcTriPoints(activeTri)"
          fill="rgba(123,31,162,0.32)"
          stroke="#7b1fa2"
          stroke-width="1.1"
          class="avg-fgc-tri-fill"
        />
        <line
          v-for="(edge, ei) in fgcTriEdges(activeTri)"
          :key="`te-${activeIndex}-${ei}`"
          :x1="fgcNodes[edge[0]].x"
          :y1="fgcNodes[edge[0]].y"
          :x2="fgcNodes[edge[1]].x"
          :y2="fgcNodes[edge[1]].y"
          stroke="#7b1fa2"
          stroke-width="1.8"
          class="avg-fgc-tri-edge"
        />
        <circle
          v-for="vi in activeTri"
          :key="`tv-${activeIndex}-${vi}`"
          :cx="fgcNodes[vi].x"
          :cy="fgcNodes[vi].y"
          r="2.7"
          fill="#7b1fa2"
          stroke="#6a1b9a"
          stroke-width="0.65"
          class="avg-fgc-tri-node"
        />
      </g>
    </svg>
  </div>
</template>
