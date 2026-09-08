<template>
  <div class="slidev-custom-brand" aria-hidden="true">
    <svg class="slidev-custom-logo" viewBox="0 0 32 32" xmlns="http://www.w3.org/2000/svg">
      <circle cx="16" cy="16" r="14" fill="none" stroke="currentColor" stroke-width="2" />
      <circle cx="16" cy="16" r="5" fill="currentColor" />
      <ellipse cx="16" cy="16" rx="14" ry="5" fill="none" stroke="currentColor" stroke-width="1.5" transform="rotate(-24 16 16)" />
    </svg>
    <span class="slidev-custom-label">Slidev</span>
  </div>
</template>

<style scoped>
.slidev-custom-brand {
  display: inline-flex;
  align-items: center;
  gap: 0.35rem;
  order: -100;
  padding-right: 0.5rem;
  margin-right: 0.25rem;
  border-right: 1px solid rgb(0 0 0 / 12%);
  color: var(--slidev-theme-primary, #1976d2);
  pointer-events: none;
  user-select: none;
}

.slidev-custom-logo {
  width: 1.35rem;
  height: 1.35rem;
  flex-shrink: 0;
}

.slidev-custom-label {
  font-size: 0.72rem;
  font-weight: 700;
  letter-spacing: 0.02em;
  line-height: 1;
  opacity: 0.85;
}
</style>
