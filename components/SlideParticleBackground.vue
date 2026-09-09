<script setup lang="ts">
import { nextTick, onMounted, onUnmounted, ref } from 'vue'

withDefaults(defineProps<{
  density?: number
}>(), {
  density: 0.7,
})

interface MatrixPanel {
  x: number
  y: number
  vx: number
  vy: number
  n: number
  cell: number
  alpha: number
  values: number[][]
  scan: number
  noise: { t: number, i: number, j: number, v: number }[]
}

interface Ball {
  x: number
  y: number
  r: number
  vx: number
  vy: number
}

const root = ref<HTMLElement | null>(null)
const canvas = ref<HTMLCanvasElement | null>(null)

let rafId = 0
let resizeObs: ResizeObserver | null = null
let w = 0
let h = 0
let ctx: CanvasRenderingContext2D | null = null

let lineRgb = '180, 83, 9'
let textRgb = '146, 64, 14'
let noiseRgb = '194, 24, 91'

let mats: MatrixPanel[] = []
let balls: Ball[] = []
let latOx = 0
let latOy = 0
let latCols = 0
let latRows = 0
let latGap = 26

function parseHexColor(input: string) {
  const hex = input.trim().replace('#', '')
  if (hex.length === 3) {
    return {
      r: Number.parseInt(hex[0] + hex[0], 16),
      g: Number.parseInt(hex[1] + hex[1], 16),
      b: Number.parseInt(hex[2] + hex[2], 16),
    }
  }
  if (hex.length === 6) {
    return {
      r: Number.parseInt(hex.slice(0, 2), 16),
      g: Number.parseInt(hex.slice(2, 4), 16),
      b: Number.parseInt(hex.slice(4, 6), 16),
    }
  }
  return { r: 245, g: 158, b: 11 }
}

function readColors() {
  const el = root.value?.parentElement
  if (!el)
    return
  const styles = getComputedStyle(el)
  const highlight = styles.getPropertyValue('--neversink-highlight-color') || '#f59e0b'
  const rgb = parseHexColor(highlight)
  lineRgb = `${rgb.r}, ${rgb.g}, ${rgb.b}`
  textRgb = `${Math.max(rgb.r - 40, 40)}, ${Math.max(rgb.g - 50, 30)}, ${Math.max(rgb.b - 20, 10)}`
}

function isRenderable() {
  const host = root.value
  if (!host)
    return false
  const rect = host.getBoundingClientRect()
  return rect.width > 8 && rect.height > 8
}

function seeded(i: number, salt: number) {
  const x = Math.sin(i * 12.9898 + salt * 78.233) * 43758.5453
  return x - Math.floor(x)
}

function makeValues(n: number, salt: number) {
  return Array.from({ length: n }, (_, i) =>
    Array.from({ length: n }, (_, j) => Math.floor(seeded(i * n + j, salt) * 5)),
  )
}

function initScene() {
  const cell = Math.max(13, Math.min(20, Math.round(Math.min(w, h) / 36)))
  mats = [
    { x: w * 0.07, y: h * 0.16, vx: 0.07, vy: 0.035, n: 6, cell, alpha: 0.42, values: makeValues(6, 3), scan: 0, noise: [] },
    { x: w * 0.64, y: h * 0.1, vx: -0.05, vy: 0.04, n: 5, cell: cell * 0.92, alpha: 0.34, values: makeValues(5, 8), scan: 11, noise: [] },
    { x: w * 0.38, y: h * 0.58, vx: 0.045, vy: -0.03, n: 7, cell: cell * 0.82, alpha: 0.28, values: makeValues(7, 14), scan: 19, noise: [] },
  ]
  latGap = Math.max(22, Math.min(32, Math.round(Math.min(w, h) / 28)))
  latOx = 18
  latOy = 16
  latCols = Math.ceil((w - latOx * 2) / latGap) + 1
  latRows = Math.ceil((h - latOy * 2) / latGap) + 1
  balls = [
    { x: w * 0.72, y: h * 0.38, r: Math.min(w, h) * 0.13, vx: 0.11, vy: 0.06 },
    { x: w * 0.28, y: h * 0.68, r: Math.min(w, h) * 0.1, vx: -0.07, vy: 0.09 },
  ]
}

function resize() {
  const el = canvas.value
  const host = root.value
  if (!el || !host)
    return

  const rect = host.getBoundingClientRect()
  const dpr = Math.min(window.devicePixelRatio || 1, 2)
  w = Math.max(Math.floor(rect.width), 1)
  h = Math.max(Math.floor(rect.height), 1)
  el.width = Math.floor(w * dpr)
  el.height = Math.floor(h * dpr)
  el.style.width = `${w}px`
  el.style.height = `${h}px`
  ctx = el.getContext('2d')
  if (!ctx)
    return
  ctx.setTransform(dpr, 0, 0, dpr, 0, 0)
  initScene()
}

function wrap(v: number, min: number, max: number) {
  const span = max - min
  if (span <= 0)
    return v
  while (v < min)
    v += span
  while (v > max)
    v -= span
  return v
}

function drawLattice(c: CanvasRenderingContext2D) {
  for (let i = 0; i < latCols; i++) {
    for (let j = 0; j < latRows; j++) {
      const x = latOx + i * latGap
      const y = latOy + j * latGap
      let inside = false
      for (const b of balls) {
        const dx = x - b.x
        const dy = y - b.y
        if (dx * dx + dy * dy <= b.r * b.r) {
          inside = true
          break
        }
      }
      if (inside) {
        c.fillStyle = `rgba(${noiseRgb}, 0.55)`
        c.beginPath()
        c.arc(x, y, 1.7, 0, Math.PI * 2)
        c.fill()
      }
      else {
        c.fillStyle = `rgba(${lineRgb}, 0.22)`
        c.beginPath()
        c.arc(x, y, 1.15, 0, Math.PI * 2)
        c.fill()
      }
    }
  }
}

function drawBalls(c: CanvasRenderingContext2D) {
  for (const b of balls) {
    c.beginPath()
    c.arc(b.x, b.y, b.r, 0, Math.PI * 2)
    c.fillStyle = `rgba(${noiseRgb}, 0.05)`
    c.fill()
    c.strokeStyle = `rgba(${noiseRgb}, 0.28)`
    c.lineWidth = 1.1
    c.stroke()
  }
}

function drawMatrix(c: CanvasRenderingContext2D, m: MatrixPanel, t: number) {
  const size = m.n * m.cell
  const si = Math.floor(m.scan / m.n) % m.n
  const sj = m.scan % m.n

  c.save()
  c.translate(m.x, m.y)
  c.globalAlpha = m.alpha

  c.fillStyle = `rgba(${lineRgb}, 0.045)`
  c.fillRect(-2, -2, size + 4, size + 4)

  c.fillStyle = `rgba(${lineRgb}, 0.08)`
  c.fillRect(0, si * m.cell, size, m.cell)
  c.fillRect(sj * m.cell, 0, m.cell, size)

  c.strokeStyle = `rgba(${lineRgb}, 0.35)`
  c.lineWidth = 0.7
  c.strokeRect(0, 0, size, size)
  c.beginPath()
  for (let k = 1; k < m.n; k++) {
    const p = k * m.cell
    c.moveTo(p, 0)
    c.lineTo(p, size)
    c.moveTo(0, p)
    c.lineTo(size, p)
  }
  c.stroke()

  c.font = `500 ${Math.max(8, m.cell * 0.42)}px "Fira Code", ui-monospace, monospace`
  c.textAlign = 'center'
  c.textBaseline = 'middle'

  for (let i = 0; i < m.n; i++) {
    for (let j = 0; j < m.n; j++) {
      const hit = m.noise.find(e => e.i === i && e.j === j && t < e.t)
      const cx = j * m.cell + m.cell / 2
      const cy = i * m.cell + m.cell / 2
      if (hit) {
        c.fillStyle = `rgba(${noiseRgb}, 0.22)`
        c.fillRect(j * m.cell + 0.5, i * m.cell + 0.5, m.cell - 1, m.cell - 1)
        c.fillStyle = `rgba(${noiseRgb}, 0.85)`
        c.fillText(String(hit.v), cx, cy)
      }
      else {
        const accent = i === si && j === sj
        c.fillStyle = accent
          ? `rgba(${textRgb}, 0.95)`
          : `rgba(${textRgb}, 0.62)`
        c.fillText(String(m.values[i]![j]!), cx, cy)
      }
    }
  }

  c.restore()
}

function step(t: number, dt: number) {
  for (const m of mats) {
    m.x += m.vx
    m.y += m.vy
    const size = m.n * m.cell
    m.x = wrap(m.x, -size, w + 8)
    m.y = wrap(m.y, -size, h + 8)
    if (Math.floor(t * 2.4 + m.n) !== Math.floor((t - dt) * 2.4 + m.n))
      m.scan = (m.scan + 1) % (m.n * m.n)
    m.noise = m.noise.filter(e => t < e.t)
    if (Math.floor(t * 0.85 + m.n) !== Math.floor((t - dt) * 0.85 + m.n) && m.noise.length < 4) {
      const tick = Math.floor(t * 11)
      const i = Math.floor(seeded(tick, m.n) * m.n)
      const j = Math.floor(seeded(tick, m.n + 5) * m.n)
      const v = Math.floor(seeded(tick, 21) * 5)
      m.noise.push({ t: t + 1.35, i, j, v })
    }
  }

  for (const b of balls) {
    b.x += b.vx
    b.y += b.vy
    if (b.x < -b.r)
      b.x = w + b.r
    if (b.x > w + b.r)
      b.x = -b.r
    if (b.y < -b.r)
      b.y = h + b.r
    if (b.y > h + b.r)
      b.y = -b.r
  }
}

let lastT = 0

function draw(now: number) {
  rafId = requestAnimationFrame(draw)

  if (!isRenderable() || !ctx || document.hidden)
    return

  const t = now / 1000
  const dt = lastT === 0 ? 0.016 : Math.min(0.05, t - lastT)
  lastT = t

  ctx.clearRect(0, 0, w, h)
  step(t, dt)
  drawLattice(ctx)
  drawBalls(ctx)
  for (const m of mats)
    drawMatrix(ctx, m, t)
}

function start() {
  readColors()
  resize()
  lastT = 0
  cancelAnimationFrame(rafId)
  rafId = requestAnimationFrame(draw)
}

function stop() {
  cancelAnimationFrame(rafId)
}

onMounted(async () => {
  if (!root.value)
    return

  await nextTick()
  requestAnimationFrame(() => start())

  resizeObs = new ResizeObserver(() => {
    if (isRenderable())
      resize()
  })
  resizeObs.observe(root.value)
})

onUnmounted(() => {
  stop()
  resizeObs?.disconnect()
})
</script>

<template>
  <div ref="root" class="slide-particle-bg" aria-hidden="true">
    <div class="slide-particle-bg-gradient" />
    <div class="slide-particle-bg-grid" />
    <canvas ref="canvas" class="slide-particle-bg-net" />
  </div>
</template>
