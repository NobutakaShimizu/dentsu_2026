<script setup lang="ts">
import { useNav, useSlideContext } from '@slidev/client'
import { computed, onUnmounted, ref, watch } from 'vue'

const STEP_MS = 420
const LOOK_MS = 520

const { $clicks } = useSlideContext()
const { isPrintMode } = useNav()

const showPairs = ref(false)
const showAlgo = ref(false)
/** 'none' | 'left' | 'right' | 'both' */
const gaze = ref<'none' | 'left' | 'right' | 'both'>('none')
const showBubble = ref(false)

let token = 0
let sequenceStarted = false

function delay(ms: number, t: number) {
  return new Promise<void>((resolve) => {
    window.setTimeout(() => {
      if (t === token)
        resolve()
    }, ms)
  })
}

function resetAll() {
  token += 1
  sequenceStarted = false
  showPairs.value = false
  showAlgo.value = false
  gaze.value = 'none'
  showBubble.value = false
}

function applyPrintFinalState() {
  token += 1
  sequenceStarted = true
  showPairs.value = true
  showAlgo.value = true
  gaze.value = 'both'
  showBubble.value = true
}

async function runRevealSequence() {
  if (isPrintMode.value) {
    applyPrintFinalState()
    return
  }

  const t = ++token
  showPairs.value = true
  showAlgo.value = false
  gaze.value = 'none'
  showBubble.value = false

  await delay(STEP_MS, t)
  if (t !== token)
    return
  showAlgo.value = true

  await delay(STEP_MS, t)
  if (t !== token)
    return
  gaze.value = 'left'

  await delay(LOOK_MS, t)
  if (t !== token)
    return
  gaze.value = 'right'

  await delay(LOOK_MS, t)
  if (t !== token)
    return
  gaze.value = 'left'

  await delay(LOOK_MS * 0.85, t)
  if (t !== token)
    return
  gaze.value = 'both'

  await delay(STEP_MS, t)
  if (t !== token)
    return
  showBubble.value = true
}

const lookingLeft = computed(() => gaze.value === 'left' || gaze.value === 'both')
const lookingRight = computed(() => gaze.value === 'right' || gaze.value === 'both')

watch(
  [$clicks, isPrintMode],
  ([current]) => {
    if (isPrintMode.value) {
      if ((current ?? 0) >= 1)
        applyPrintFinalState()
      else
        resetAll()
      return
    }

    if ((current ?? 0) < 1) {
      resetAll()
      return
    }

    if (!sequenceStarted) {
      sequenceStarted = true
      runRevealSequence()
    }
  },
  { immediate: true },
)

onUnmounted(() => {
  resetAll()
})
</script>

<template>
  <div class="ppi-root">
    <div class="ppi" v-click>
      <div class="ppi-stage">
        <!-- Left: true randomness -->
        <div
          class="ppi-pair ppi-pair--true"
          :class="{ 'is-on': showPairs, 'is-gazed': lookingLeft }"
        >
          <div class="ppi-pair-body">
            <svg
              class="ppi-pair-icon ppi-pair-icon--dice"
              viewBox="0 0 48 48"
              aria-hidden="true"
            >
              <rect
                x="8"
                y="8"
                width="32"
                height="32"
                rx="6"
                ry="6"
                fill="#fff3e0"
                stroke="#e65100"
                stroke-width="2"
              />
              <circle cx="18" cy="18" r="2.8" fill="#e65100" />
              <circle cx="30" cy="18" r="2.8" fill="#e65100" />
              <circle cx="24" cy="24" r="2.8" fill="#e65100" />
              <circle cx="18" cy="30" r="2.8" fill="#e65100" />
              <circle cx="30" cy="30" r="2.8" fill="#e65100" />
            </svg>
            <div class="ppi-pair-content">
              <div class="ppi-pair-title">
                <Math tex="(X, Y)" />
              </div>
              <div class="ppi-pair-notes">
                <span class="ppi-pair-note">
                  <Math tex="X" />: 一様ランダムな文字列
                </span>
                <span class="ppi-pair-note">
                  <Math tex="Y" />: ある程度のエントロピー
                </span>
              </div>
            </div>
          </div>
        </div>

        <!-- Center: efficient algorithm -->
        <div class="ppi-center" :class="{ 'is-on': showAlgo }">
          <svg
            class="ppi-gaze-svg"
            viewBox="0 0 280 72"
            preserveAspectRatio="none"
            aria-hidden="true"
          >
            <path
              class="ppi-gaze-path"
              :class="{ 'is-on': lookingLeft }"
              d="M140 36 C100 36, 70 28, 36 24"
              fill="none"
              stroke-width="2"
              stroke-linecap="round"
              stroke-dasharray="4 3"
            />
            <path
              class="ppi-gaze-path"
              :class="{ 'is-on': lookingRight }"
              d="M140 36 C180 36, 210 28, 244 24"
              fill="none"
              stroke-width="2"
              stroke-linecap="round"
              stroke-dasharray="4 3"
            />
            <circle
              class="ppi-gaze-dot"
              :class="{ 'is-on': lookingLeft }"
              cx="30"
              cy="23"
              r="3.2"
            />
            <circle
              class="ppi-gaze-dot"
              :class="{ 'is-on': lookingRight }"
              cx="250"
              cy="23"
              r="3.2"
            />
          </svg>

          <div
            class="ppi-person"
            :class="{
              'look-left': gaze === 'left',
              'look-right': gaze === 'right',
              'look-both': gaze === 'both',
            }"
          >
            <PrgPerson pose="observer" />
            <span class="ppi-person-label">効率的アルゴリズム</span>
          </div>

          <div class="ppi-bubble" :class="{ 'is-on': showBubble }">
            <span class="ppi-bubble-text">
              双方を見ても
              <br>識別できない！
            </span>
          </div>
        </div>

        <!-- Right: pseudorandom -->
        <div
          class="ppi-pair ppi-pair--pr"
          :class="{ 'is-on': showPairs, 'is-gazed': lookingRight }"
        >
          <div class="ppi-pair-body">
            <svg
              class="ppi-pair-icon ppi-pair-icon--machine"
              viewBox="0 0 48 48"
              aria-hidden="true"
            >
              <rect
                x="11"
                y="18"
                width="26"
                height="18"
                rx="3"
                fill="#e3f2fd"
                stroke="#1565c0"
                stroke-width="2"
              />
              <rect x="15" y="22" width="7" height="5" rx="1" fill="#1565c0" opacity="0.75" />
              <rect x="26" y="22" width="7" height="5" rx="1" fill="#1565c0" opacity="0.75" />
              <line x1="22.5" y1="24.5" x2="25.5" y2="24.5" stroke="#1565c0" stroke-width="1.8" stroke-linecap="round" />
              <polygon points="25.5,24.5 23.8,23.4 23.8,25.6" fill="#1565c0" />
              <line x1="5" y1="27" x2="11" y2="27" stroke="#1565c0" stroke-width="2" stroke-linecap="round" />
              <polygon points="5,27 9,24.5 9,29.5" fill="#1565c0" />
              <line x1="37" y1="27" x2="43" y2="27" stroke="#1565c0" stroke-width="2" stroke-linecap="round" />
              <polygon points="43,27 39,24.5 39,29.5" fill="#1565c0" />
              <g transform="translate(24, 10)">
                <circle r="6.5" fill="#fff" stroke="#1565c0" stroke-width="2" />
                <circle r="2.2" fill="#1565c0" />
                <rect x="-1.1" y="-8.2" width="2.2" height="2.8" rx="0.4" fill="#1565c0" />
                <rect x="-1.1" y="5.4" width="2.2" height="2.8" rx="0.4" fill="#1565c0" />
                <rect x="-8.2" y="-1.1" width="2.8" height="2.2" rx="0.4" fill="#1565c0" />
                <rect x="5.4" y="-1.1" width="2.8" height="2.2" rx="0.4" fill="#1565c0" />
                <rect x="-5.8" y="-5.8" width="2.2" height="2.8" rx="0.4" fill="#1565c0" transform="rotate(-45)" />
                <rect x="3.6" y="3.6" width="2.2" height="2.8" rx="0.4" fill="#1565c0" transform="rotate(-45)" />
                <rect x="3.6" y="-5.8" width="2.2" height="2.8" rx="0.4" fill="#1565c0" transform="rotate(45)" />
                <rect x="-5.8" y="3.6" width="2.2" height="2.8" rx="0.4" fill="#1565c0" transform="rotate(45)" />
              </g>
              <rect x="17" y="36" width="4" height="5" rx="1" fill="#1565c0" opacity="0.65" />
              <rect x="27" y="36" width="4" height="5" rx="1" fill="#1565c0" opacity="0.65" />
            </svg>
            <div class="ppi-pair-content">
              <div class="ppi-pair-title">
                <Math tex="(X, f(X))" />
              </div>
              <div class="ppi-pair-notes">
                <span class="ppi-pair-note">
                  <Math tex="X" />: 一様ランダムな文字列
                </span>
                <span class="ppi-pair-note">
                  <Math tex="f(X)" />: 計算量的疑似ランダム
                </span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.ppi-root {
  position: relative;
  width: 100%;
}

.ppi {
  margin-top: 0.55rem;
  width: 100%;
  padding: 0 0.2rem;
  box-sizing: border-box;
}

.ppi-stage {
  display: grid;
  grid-template-columns: minmax(11rem, 1fr) minmax(10rem, 1.15fr) minmax(11rem, 1fr);
  align-items: center;
  gap: 0.75rem 1.4rem;
  width: 100%;
  min-height: 8.2rem;
}

.ppi-pair {
  display: flex;
  flex-direction: column;
  align-items: stretch;
  padding: 0.55rem 0.85rem 0.55rem 0.65rem;
  border-radius: 8px;
  border: 1.5px solid rgba(245, 124, 0, 0.45);
  background: linear-gradient(135deg, #fffdf8 0%, #fffaf3 100%);
  opacity: 0;
  transition:
    opacity 0.35s ease,
    transform 0.35s ease,
    box-shadow 0.35s ease,
    border-color 0.35s ease;
  min-width: 0;
}

.ppi-pair--true {
  transform: translateX(-14px);
  justify-self: start;
  width: 100%;
  max-width: 16.5rem;
}

.ppi-pair--pr {
  transform: translateX(14px);
  justify-self: end;
  width: 100%;
  max-width: 16.5rem;
  border-color: rgba(25, 118, 210, 0.4);
  background: linear-gradient(135deg, #f5faff 0%, #eef5fc 100%);
}

.ppi-pair.is-on {
  opacity: 1;
  transform: translateX(0);
}

.ppi-pair.is-gazed {
  box-shadow: 0 0 0 2px rgba(239, 108, 0, 0.28), 0 4px 14px rgba(239, 108, 0, 0.12);
}

.ppi-pair--pr.is-gazed {
  box-shadow: 0 0 0 2px rgba(21, 101, 192, 0.28), 0 4px 14px rgba(21, 101, 192, 0.12);
}

.ppi-pair-body {
  display: flex;
  align-items: center;
  gap: 0.55rem;
}

.ppi-pair-icon {
  width: 2.35rem;
  height: 2.35rem;
  flex-shrink: 0;
}

.ppi-pair-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.2rem;
  flex: 1;
  min-width: 0;
}

.ppi-pair-title {
  font-weight: 700;
  color: #bf360c;
  line-height: 1.2;
}

.ppi-pair--pr .ppi-pair-title {
  color: #1565c0;
}

.ppi-pair-title :deep(.katex) {
  font-size: 1.15rem;
  font-weight: 700;
}

.ppi-pair-notes {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.06rem;
  font-size: 0.62rem;
  font-weight: 600;
  color: #78909c;
  letter-spacing: 0.01em;
  line-height: 1.25;
}

.ppi-pair-note {
  display: inline-flex;
  align-items: baseline;
  gap: 0.08em;
}

.ppi-pair-note :deep(.katex) {
  font-size: 0.95em;
  font-weight: 700;
}

.ppi-center {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 7.5rem;
  opacity: 0;
  transform: translateY(8px) scale(0.96);
  transition: opacity 0.35s ease, transform 0.35s ease;
}

.ppi-center.is-on {
  opacity: 1;
  transform: translateY(0) scale(1);
}

.ppi-gaze-svg {
  position: absolute;
  left: 50%;
  top: 1.1rem;
  width: min(100%, 18rem);
  height: 3.2rem;
  transform: translateX(-50%);
  overflow: visible;
  pointer-events: none;
}

.ppi-gaze-path {
  stroke: #ef6c00;
  opacity: 0;
  transition: opacity 0.28s ease;
}

.ppi-gaze-path.is-on {
  opacity: 0.85;
}

.ppi-gaze-dot {
  fill: #ef6c00;
  opacity: 0;
  transition: opacity 0.28s ease;
}

.ppi-gaze-dot.is-on {
  opacity: 1;
}

.ppi-person {
  position: relative;
  z-index: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.12rem;
  transition: transform 0.35s ease;
}

.ppi-person.look-left {
  transform: translateX(-4px);
}

.ppi-person.look-right {
  transform: translateX(4px);
}

.ppi-person.look-both {
  transform: translateX(0);
}

.ppi-person :deep(.prg-person-svg) {
  width: 3.1rem;
  height: auto;
  transition: transform 0.35s ease;
}

/* PrgPerson observer は既定で左向き */
.ppi-person.look-left :deep(.prg-person-svg) {
  transform: scaleX(1);
}

.ppi-person.look-right :deep(.prg-person-svg) {
  transform: scaleX(-1);
}

.ppi-person.look-both :deep(.prg-person-svg) {
  transform: scaleX(1);
}

.ppi-person-label {
  font-size: 0.68rem;
  font-weight: 700;
  color: #ef6c00;
  letter-spacing: 0.02em;
  white-space: nowrap;
}

.ppi-bubble {
  position: relative;
  z-index: 2;
  margin-top: 0.35rem;
  min-width: 7.2rem;
  padding: 0.38rem 0.55rem;
  border-radius: 14px;
  background: #e8f5e9;
  border: 1.5px solid #66bb6a;
  box-shadow: 0 2px 8px rgba(46, 125, 50, 0.12);
  opacity: 0;
  transform: translateY(8px) scale(0.92);
  transition: opacity 0.35s ease, transform 0.35s ease;
}

.ppi-bubble.is-on {
  opacity: 1;
  transform: translateY(0) scale(1);
}

.ppi-bubble::before {
  content: '';
  position: absolute;
  left: 50%;
  top: -9px;
  transform: translateX(-50%);
  border-style: solid;
  border-width: 0 7px 9px 7px;
  border-color: transparent transparent #66bb6a transparent;
}

.ppi-bubble::after {
  content: '';
  position: absolute;
  left: 50%;
  top: -6.5px;
  transform: translateX(-50%);
  border-style: solid;
  border-width: 0 5.5px 7px 5.5px;
  border-color: transparent transparent #e8f5e9 transparent;
}

.ppi-bubble-text {
  display: block;
  font-size: 0.82rem;
  font-weight: 800;
  color: #1b5e20;
  line-height: 1.35;
  text-align: center;
}
</style>
