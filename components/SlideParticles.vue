<script setup lang="ts">
const particles = Array.from({ length: 48 }, (_, i) => {
  const kind = i % 3
  return {
    id: i,
    left: `${(i * 17 + 3) % 96 + 2}%`,
    top: `${(i * 23 + 9) % 90 + 5}%`,
    size: kind === 0 ? 5 + (i % 3) : kind === 1 ? 3 + (i % 2) : 10 + (i % 4),
    duration: 16 + (i % 9) * 2,
    delay: -(i % 14),
    opacity: kind === 2 ? 0.14 + (i % 4) * 0.04 : 0.38 + (i % 5) * 0.08,
    drift: i % 3,
  }
})
</script>

<template>
  <div class="slide-particles" aria-hidden="true">
    <span
      v-for="p in particles"
      :key="p.id"
      class="slide-particle"
      :class="[
        `slide-particle-drift-${p.drift}`,
        { 'slide-particle-soft': p.size >= 10 },
      ]"
      :style="{
        left: p.left,
        top: p.top,
        width: `${p.size}px`,
        height: `${p.size}px`,
        opacity: p.opacity,
        animationDuration: `${p.duration}s`,
        animationDelay: `${p.delay}s`,
      }"
    />
  </div>
</template>

<style scoped>
.slide-particles {
  position: absolute;
  inset: 0;
  overflow: hidden;
  pointer-events: none;
  z-index: 1;
}

.slide-particle {
  position: absolute;
  border-radius: 50%;
  background: color-mix(
    in srgb,
    var(--neversink-highlight-color, #1976d2) 55%,
    var(--neversink-text-color, #333) 45%
  );
  box-shadow:
    0 0 8px color-mix(in srgb, var(--neversink-highlight-color, #1976d2) 50%, transparent),
    0 0 2px color-mix(in srgb, var(--neversink-text-color, #333) 25%, transparent);
  animation-timing-function: ease-in-out;
  animation-iteration-count: infinite;
}

.slide-particle-soft {
  background: color-mix(
    in srgb,
    var(--neversink-highlight-color, #1976d2) 22%,
    transparent
  );
  box-shadow: 0 0 18px color-mix(in srgb, var(--neversink-highlight-color, #1976d2) 35%, transparent);
  filter: blur(1px);
}

.slide-particle-drift-0 {
  animation-name: slide-particle-drift-a;
}

.slide-particle-drift-1 {
  animation-name: slide-particle-drift-b;
}

.slide-particle-drift-2 {
  animation-name: slide-particle-drift-c;
}

@keyframes slide-particle-drift-a {
  0%, 100% { transform: translate(0, 0) scale(1); }
  33% { transform: translate(18px, -24px) scale(1.08); }
  66% { transform: translate(-12px, -16px) scale(0.95); }
}

@keyframes slide-particle-drift-b {
  0%, 100% { transform: translate(0, 0) scale(1); }
  40% { transform: translate(-20px, -20px) scale(1.05); }
  70% { transform: translate(14px, -30px) scale(0.92); }
}

@keyframes slide-particle-drift-c {
  0%, 100% { transform: translate(0, 0) scale(1); }
  50% { transform: translate(10px, 16px) scale(1.12); }
}
</style>
