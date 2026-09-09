<script setup lang="ts">
import { computed, ref, watch } from 'vue'
import { useNav, useSlideContext } from '@slidev/client'
import MathTex from './Math.vue'

const X = [2, 0, 1, 3]
const Y = [2, 0, 1, 3, 4, 1, 2, 0]
const YTILDE = [2, 0, 4, 3, 4, 0, 2, 0]
const NOISE = new Set([2, 5])

const { $clicks } = useSlideContext()
const { isPrintMode } = useNav()

const clicks = computed(() => $clicks.value ?? 0)
const stage = computed(() => (isPrintMode.value ? 3 : clicks.value))

const showX = ref(true)
const showEnc = ref(false)
const showY = ref(false)
const showNoise = ref(false)
const showYtilde = ref(false)
const showFlips = ref(false)
const showDec = ref(false)
const showXd = ref(false)
const showHint = ref(false)

watch(
  stage,
  (n) => {
    const s = isPrintMode.value ? 3 : n
    showX.value = true
    showEnc.value = s >= 1
    showY.value = s >= 1
    showNoise.value = s >= 2
    showYtilde.value = s >= 2
    showFlips.value = s >= 2
    showDec.value = s >= 3
    showXd.value = s >= 3
    showHint.value = s >= 3
  },
  { immediate: true },
)
</script>

<template>
  <div class="ecc">
    <div v-click class="ecc-click" aria-hidden="true" />
    <div v-click class="ecc-click" aria-hidden="true" />
    <div v-click class="ecc-click" aria-hidden="true" />

    <div class="ecc-x1">
      <div class="ecc-block" :class="{ 'is-in': showX }">
        <div class="ecc-name"><MathTex tex="x" /></div>
        <div class="ecc-vec">
          <div v-for="(v, i) in X" :key="`x-${i}`" class="ecc-cell">{{ v }}</div>
        </div>
      </div>
    </div>

    <div class="ecc-harrow" :class="{ 'is-in': showEnc }">
      <span class="ecc-harrow-label"><MathTex tex="\mathrm{Enc}" /></span>
      <span class="ecc-harrow-shaft"><span class="ecc-harrow-line" /></span>
    </div>

    <div class="ecc-y">
      <div class="ecc-block" :class="{ 'is-in': showY }">
        <div class="ecc-name"><MathTex tex="y=\mathrm{Enc}(x)" /></div>
        <div class="ecc-vec">
          <div v-for="(v, i) in Y" :key="`y-${i}`" class="ecc-cell">{{ v }}</div>
        </div>
      </div>
    </div>

    <div class="ecc-varrow" :class="{ 'is-in': showNoise }">
      <span class="ecc-varrow-label">通信時のノイズ</span>
      <span class="ecc-varrow-line" />
    </div>

    <div class="ecc-xd">
      <div class="ecc-block" :class="{ 'is-in': showXd }">
        <div class="ecc-name"><MathTex tex="x" /></div>
        <div class="ecc-vec is-clean">
          <div v-for="(v, i) in X" :key="`xd-${i}`" class="ecc-cell is-clean">{{ v }}</div>
        </div>
      </div>
    </div>

    <div class="ecc-harrow ecc-harrow-rev" :class="{ 'is-in': showDec }">
      <span class="ecc-harrow-label"><MathTex tex="\mathrm{Dec}" /></span>
      <span class="ecc-harrow-shaft"><span class="ecc-harrow-line" /></span>
    </div>

    <div class="ecc-yt">
      <div class="ecc-block" :class="{ 'is-in': showYtilde }">
        <div class="ecc-name"><MathTex tex="\widetilde{y}" /></div>
        <div class="ecc-vec">
          <div
            v-for="(v, i) in YTILDE"
            :key="`yt-${i}`"
            class="ecc-cell"
            :class="{ 'is-noise': showFlips && NOISE.has(i) }"
          >{{ v }}</div>
        </div>
        <div class="ecc-note" :class="{ 'is-in': showFlips }">いくつかの要素が変化する</div>
      </div>
    </div>

    <p class="ecc-hint" :class="{ 'is-in': showHint }">
      <MathTex tex="\mathrm{dist}(y,\widetilde{y})" /> が小さければ…
    </p>
  </div>
</template>

<style scoped>
.ecc {
  position: relative;
  display: grid;
  grid-template-columns: max-content 8.2rem max-content;
  grid-template-rows: auto 2.6rem auto auto;
  align-items: center;
  justify-content: center;
  column-gap: 0.4rem;
  row-gap: 0.2rem;
  margin: 0.5rem auto 0.15rem;
  width: fit-content;
}

.ecc-click {
  position: absolute;
  width: 0;
  height: 0;
  overflow: hidden;
}

.ecc-x1 { grid-column: 1; grid-row: 1; }
.ecc-harrow { grid-column: 2; grid-row: 1; }
.ecc-y { grid-column: 3; grid-row: 1; }
.ecc-varrow { grid-column: 3; grid-row: 2; justify-self: center; }
.ecc-xd { grid-column: 1; grid-row: 3; }
.ecc-harrow-rev { grid-column: 2; grid-row: 3; }
.ecc-yt { grid-column: 3; grid-row: 3; }
.ecc-hint { grid-column: 1 / 3; grid-row: 4; }

.ecc-block {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.18rem;
  opacity: 0;
  transform: translateY(8px);
}

.ecc-block.is-in {
  opacity: 1;
  transform: none;
  transition: opacity 0.28s ease, transform 0.28s ease;
}

.ecc-name {
  font-size: 0.95rem;
  font-weight: 700;
  color: #37474f;
  line-height: 1.15;
  min-height: 1.15rem;
}

.ecc-vec {
  display: flex;
  gap: 1px;
  background: #b0bec5;
  border: 1px solid #78909c;
  padding: 1px;
}

.ecc-vec.is-clean {
  border-color: #90a4ae;
}

.ecc-cell {
  width: 18px;
  height: 18px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #fff;
  color: #455a64;
  font-family: 'Fira Code', monospace;
  font-size: 0.72rem;
  line-height: 1;
  transition: background-color 0.2s ease, color 0.2s ease;
}

.ecc-cell.is-noise {
  background: #f8bbd0;
  color: #880e4f;
  font-weight: 700;
}

.ecc-cell.is-clean {
  color: #37474f;
}

.ecc-note {
  font-family: 'Roboto', sans-serif;
  font-size: 0.68rem;
  font-weight: 600;
  color: #c2185b;
  opacity: 0;
  min-height: 0.9rem;
}

.ecc-note.is-in {
  opacity: 1;
  transition: opacity 0.24s ease;
}

.ecc-harrow {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 100%;
  min-width: 8.2rem;
  opacity: 0;
  transform: translateX(-8px);
}

.ecc-harrow.is-in {
  opacity: 1;
  transform: none;
  transition: opacity 0.24s ease, transform 0.24s ease;
}

.ecc-harrow-label {
  font-size: 0.78rem;
  font-weight: 700;
  color: #546e7a;
  line-height: 1;
  margin-bottom: 0.1rem;
}

.ecc-harrow-shaft {
  position: relative;
  display: block;
  width: 100%;
  height: 0.7rem;
}

.ecc-harrow-line {
  position: absolute;
  left: 0.1rem;
  right: 0.1rem;
  top: 50%;
  border-top: 2px solid #546e7a;
}

.ecc-harrow-line::after {
  content: '';
  position: absolute;
  right: -1px;
  top: 50%;
  border-top: 5px solid transparent;
  border-bottom: 5px solid transparent;
  border-left: 9px solid #546e7a;
  transform: translateY(-50%);
}

.ecc-harrow-rev .ecc-harrow-line::after {
  right: auto;
  left: -1px;
  border-left: none;
  border-right: 9px solid #546e7a;
}

.ecc-varrow {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 0.12rem;
  opacity: 0;
  transform: translateY(-6px);
}

.ecc-varrow.is-in {
  opacity: 1;
  transform: none;
  transition: opacity 0.24s ease, transform 0.24s ease;
}

.ecc-varrow-label {
  font-family: 'Roboto', sans-serif;
  font-size: 0.7rem;
  font-weight: 600;
  color: #c2185b;
  white-space: nowrap;
}

.ecc-varrow-line {
  position: relative;
  display: block;
  width: 0;
  height: 1.35rem;
  border-left: 2px solid #546e7a;
}

.ecc-varrow-line::after {
  content: '';
  position: absolute;
  left: -6px;
  bottom: -2px;
  border-left: 5px solid transparent;
  border-right: 5px solid transparent;
  border-top: 9px solid #546e7a;
}

.ecc-hint {
  margin: 0.25rem 0 0;
  font-family: 'Roboto', sans-serif;
  font-size: 0.82rem;
  color: #546e7a;
  opacity: 0;
  justify-self: start;
}

.ecc-hint.is-in {
  opacity: 1;
  transition: opacity 0.24s ease;
}
</style>
