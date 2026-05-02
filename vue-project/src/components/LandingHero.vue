<template>
  <section id="s1" class="hero">
    <canvas ref="canvasEl" class="heroCanvas" aria-hidden="true"></canvas>

    <div class="heroShade"></div>

    <div class="heroContent">
      <div class="heroInner">
        <span class="kicker">New York Outdoor Dining</span>

        <h1 class="headline">
          <span class="line line1">Why does</span>
          <span class="line line2">outdoor dining</span>
          <span class="line line3">feel gone?</span>
        </h1>

        <div class="dekBlock">
          <p class="dek dek1">What remains is harder to see.</p>
          <p class="dek dek2">It did not disappear. It became uneven.</p>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup>
import { onMounted, onBeforeUnmount, ref } from 'vue'

const canvasEl = ref(null)

let animId = null
let ctx = null
let W = 0
let H = 0
let dots = []
let t = 0
let startTime = null

const COLORS = {
  bg: '#fafaf8',
  gray: '110, 106, 100',
  bandGray: '72, 68, 62',
  red: '210, 82, 62',
}

const CLUSTERS = [
  { nx: 0.64, ny: 0.42, r: 0.048 },
  { nx: 0.71, ny: 0.54, r: 0.055 },
  { nx: 0.61, ny: 0.69, r: 0.052 },
  { nx: 0.78, ny: 0.77, r: 0.044 },
]

const BANDS = [
  { x: 0.58, tilt: -0.08, width: 0.046 },
  { x: 0.68, tilt: 0.07, width: 0.052 },
  { x: 0.76, tilt: -0.035, width: 0.044 },
]

function random(min, max) {
  return min + Math.random() * (max - min)
}

function clamp(value, min, max) {
  return Math.max(min, Math.min(max, value))
}

function inCluster(nx, ny) {
  for (const c of CLUSTERS) {
    const dx = nx - c.nx
    const dy = ny - c.ny

    if (Math.sqrt(dx * dx + dy * dy) < c.r) {
      return true
    }
  }

  return false
}

function nearBand(nx, ny) {
  for (const b of BANDS) {
    const lineX = b.x + (ny - 0.5) * b.tilt

    if (Math.abs(nx - lineX) < b.width) {
      return true
    }
  }

  return false
}

function sampleBandPoint() {
  const band = BANDS[Math.floor(Math.random() * BANDS.length)]
  const ny = random(0.08, 0.94)
  const nx = band.x + (ny - 0.5) * band.tilt + random(-band.width, band.width)

  return {
    nx: clamp(nx, 0.02, 0.98),
    ny,
  }
}

function sampleFreePoint() {
  let nx = Math.random()
  let ny = Math.random()

  const inTextZone = nx < 0.52 && ny > 0.14 && ny < 0.86

  if (inTextZone && Math.random() < 0.82) {
    nx = random(0.52, 0.98)
  }

  return { nx, ny }
}

function initDots() {
  dots = []

  const count = Math.floor((W * H) / 2850)

  for (let i = 0; i < count; i += 1) {
    let point

    if (Math.random() < 0.43) {
      point = sampleBandPoint()
    } else {
      point = sampleFreePoint()
    }

    const nx = point.nx
    const ny = point.ny

    const isBand = nearBand(nx, ny)
    const isRed = inCluster(nx, ny) && Math.random() < 0.26

    dots.push({
      x: nx * W,
      y: ny * H,
      nx,
      ny,
      isRed,
      isBand,
      r: isRed
        ? random(5.0, 7.0)
        : isBand
          ? random(2.8, 4.8)
          : random(1.8, 3.4),
      alpha: isRed
        ? random(0.78, 0.94)
        : isBand
          ? random(0.22, 0.36)
          : random(0.09, 0.21),
      deathTime: isRed ? Infinity : random(1.2, 4.2),
      vx: random(-0.075, 0.075),
      vy: random(-0.075, 0.075),
      phase: random(0, Math.PI * 2),
    })
  }
}

function resize() {
  const canvas = canvasEl.value
  if (!canvas) return

  const rect = canvas.getBoundingClientRect()
  const dpr = Math.min(window.devicePixelRatio || 1, 2)

  W = rect.width * dpr
  H = rect.height * dpr

  canvas.width = W
  canvas.height = H

  ctx = canvas.getContext('2d')
  initDots()
}

function drawRedDot(d) {
  const emergeStart = 0.8
  const emergeEnd = 2.2
  const baseOpacity = 0.08

  let opacity

  if (t < emergeStart) {
    opacity = baseOpacity
  } else if (t < emergeEnd) {
    const p = (t - emergeStart) / (emergeEnd - emergeStart)
    opacity = baseOpacity + (d.alpha - baseOpacity) * p
  } else {
    const pulse = Math.sin(t * 1.05 + d.phase) * 0.10 + 0.90
    opacity = d.alpha * pulse
  }

  ctx.beginPath()
  ctx.arc(d.x, d.y, d.r, 0, Math.PI * 2)
  ctx.fillStyle = `rgba(${COLORS.red}, ${opacity})`
  ctx.fill()
}

function drawGrayDot(d) {
  const fadeStart = d.deathTime
  const fadeDur = 2.4
  const restingOpacity = d.isBand ? 0.16 : 0.09

  let opacity = d.alpha

  if (t >= fadeStart && t < fadeStart + fadeDur) {
    const p = (t - fadeStart) / fadeDur
    opacity = d.alpha * (1 - p) + restingOpacity * p
  } else if (t >= fadeStart + fadeDur) {
    opacity = restingOpacity
  }

  ctx.beginPath()
  ctx.arc(d.x, d.y, d.r, 0, Math.PI * 2)
  ctx.fillStyle = d.isBand
    ? `rgba(${COLORS.bandGray}, ${opacity})`
    : `rgba(${COLORS.gray}, ${opacity})`
  ctx.fill()
}

function draw(ts) {
  if (!ctx) return

  if (!startTime) {
    startTime = ts
  }

  t = (ts - startTime) / 1000

  ctx.clearRect(0, 0, W, H)
  ctx.fillStyle = COLORS.bg
  ctx.fillRect(0, 0, W, H)

  for (const d of dots) {
    d.x += d.vx
    d.y += d.vy

    if (d.x < -30) d.x = W + 30
    if (d.x > W + 30) d.x = -30
    if (d.y < -30) d.y = H + 30
    if (d.y > H + 30) d.y = -30

    if (d.isRed) {
      drawRedDot(d)
    } else {
      drawGrayDot(d)
    }
  }

  animId = requestAnimationFrame(draw)
}

onMounted(() => {
  resize()

  window.addEventListener('resize', resize)
  animId = requestAnimationFrame(draw)
})

onBeforeUnmount(() => {
  if (animId) {
    cancelAnimationFrame(animId)
  }

  window.removeEventListener('resize', resize)
})
</script>

<style scoped>
.hero {
  position: relative;
  width: 100%;
  min-height: 100svh;
  background: #fafaf8;
  border-bottom: 1px solid #e6e3de;
  overflow: hidden;
}

.heroCanvas {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  display: block;
}

.heroShade {
  position: absolute;
  inset: 0;
  z-index: 1;
  pointer-events: none;
  background:
    radial-gradient(
      circle at 30% 54%,
      rgba(250, 250, 248, 0.80) 0%,
      rgba(250, 250, 248, 0.66) 24%,
      rgba(250, 250, 248, 0.30) 52%,
      rgba(250, 250, 248, 0.06) 100%
    );
}

.heroContent {
  position: relative;
  z-index: 2;
  width: 100%;
  min-height: 100svh;
  display: flex;
  align-items: center;
  justify-content: flex-start;
  padding: 72px 0;
  box-sizing: border-box;
}

.heroInner {
  width: min(1080px, calc(100% - 120px));
  margin: 0 auto;
  transform: translateY(-2vh);
}

.kicker {
  display: block;
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 12px;
  font-weight: 500;
  letter-spacing: 0.14em;
  text-transform: uppercase;
  color: #9d968e;
  margin-bottom: 36px;
  opacity: 0;
  animation: fadeUp 0.6s ease forwards;
  animation-delay: 0.2s;
}

.headline {
  display: flex;
  flex-direction: column;
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: clamp(58px, 8.5vw, 132px);
  font-weight: 700;
  line-height: 0.9;
  letter-spacing: -0.052em;
  color: #111111;
  margin: 0 0 42px;
}

.line {
  display: block;
  opacity: 0;
  transform: translateY(18px);
}

.line1 {
  animation: fadeUp 0.7s ease forwards;
  animation-delay: 0.35s;
}

.line2 {
  animation: fadeUp 0.7s ease forwards;
  animation-delay: 0.5s;
}

.line3 {
  animation: fadeUp 0.7s ease forwards;
  animation-delay: 0.65s;
}

.dekBlock {
  margin: 0;
}

.dek {
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: clamp(18px, 1.7vw, 24px);
  line-height: 1.42;
  margin: 0;
  opacity: 0;
}

.dek1 {
  color: #111111;
  font-weight: 600;
  animation: fadeUp 0.6s ease forwards;
  animation-delay: 0.9s;
}

.dek2 {
  color: #746f69;
  font-weight: 400;
  margin-top: 7px;
  animation: fadeUp 0.6s ease forwards;
  animation-delay: 1.08s;
}

@keyframes fadeUp {
  from {
    opacity: 0;
    transform: translateY(14px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@media (max-width: 760px) {
  .heroContent {
    padding: 52px 0;
  }

  .heroInner {
    width: calc(100% - 56px);
    transform: translateY(-1vh);
  }

  .kicker {
    margin-bottom: 26px;
  }

  .headline {
    font-size: clamp(50px, 14vw, 82px);
    margin-bottom: 32px;
  }

  .dek {
    font-size: 18px;
  }
}
</style>