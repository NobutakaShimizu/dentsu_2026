<script setup lang="ts">
defineProps<{
  ignited: boolean
}>()
</script>

<template>
  <div
    class="suika-walker"
    :class="{ 'is-ignited': ignited }"
    aria-hidden="true"
  >
    <svg
      viewBox="0 0 48 56"
      class="suika-walker-svg"
    >
      <!-- stick -->
      <g class="suika-stick">
        <line
          x1="28"
          y1="22"
          x2="46"
          y2="6"
          class="suika-stick-shaft"
        />
        <circle
          cx="46"
          cy="5"
          r="2.2"
          class="suika-stick-tip"
        />
      </g>

      <!-- body -->
      <ellipse
        cx="18"
        cy="30"
        rx="7"
        ry="9"
        class="suika-person-body"
      />

      <!-- head + blindfold -->
      <circle
        cx="18"
        cy="16"
        r="7"
        class="suika-person-head"
      />
      <rect
        x="10"
        y="14"
        width="16"
        height="4.5"
        rx="1.2"
        class="suika-blindfold"
      />
      <path
        d="M10 16 H6"
        class="suika-blindfold-tie"
      />
      <path
        d="M26 16 H30"
        class="suika-blindfold-tie"
      />

      <!-- arms -->
      <path
        d="M22 26 C26 24 28 22 30 20"
        class="suika-person-arm"
      />

      <!-- legs (walk cycle via CSS) -->
      <g class="suika-legs">
        <path
          d="M15 38 C13 44 12 50 11 54"
          class="suika-leg suika-leg-l"
        />
        <path
          d="M21 38 C23 44 24 50 25 54"
          class="suika-leg suika-leg-r"
        />
      </g>
    </svg>
  </div>
</template>
