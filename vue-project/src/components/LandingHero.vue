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

const CLUSTERS = [
  { nx: 0.28, ny: 0.38, r: 0.08 },
  { nx: 0.38, ny: 0.48, r: 0.07 },
  { nx: 0.68, ny: 0.40, r: 0.09 },
  { nx: 0.58, ny: 0.66, r: 0.07 },
  { nx: 0.76, ny: 0.70, r: 0.06 },
]

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

function initDots() {
  dots = []

  const count = Math.floor((W * H) / 3000)

  for (let i = 0; i < count; i += 1) {
    const nx = Math.random()
    const ny = Math.random()

    const isRed = inCluster(nx, ny) && Math.random() < 0.30

    dots.push({
      x: nx * W,
      y: ny * H,
      nx,
      ny,
      r: isRed ? 4.2 + Math.random() * 1.8 : 2.4 + Math.random() * 2.0,
      isRed,
      alpha: isRed ? 0.78 + Math.random() * 0.18 : 0.30 + Math.random() * 0.22,
      deathTime: isRed ? Infinity : 1.0 + Math.random() * 3.0,
      vx: (Math.random() - 0.5) * 0.18,
      vy: (Math.random() - 0.5) * 0.18,
      phase: Math.random() * Math.PI * 2,
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

function draw(ts) {
  if (!ctx) return

  if (!startTime) {
    startTime = ts
  }

  t = (ts - startTime) / 1000

  ctx.clearRect(0, 0, W, H)
  ctx.fillStyle = '#fafaf8'
  ctx.fillRect(0, 0, W, H)

  for (const d of dots) {
    d.x += d.vx
    d.y += d.vy

    if (d.x < -20) d.x = W + 20
    if (d.x > W + 20) d.x = -20
    if (d.y < -20) d.y = H + 20
    if (d.y > H + 20) d.y = -20

    if (d.isRed) {
      const emergeStart = 1.0
      const emergeEnd = 2.4

      let opacity = 0

      if (t < emergeStart) {
        opacity = 0.10
      } else if (t < emergeEnd) {
        const p = (t - emergeStart) / (emergeEnd - emergeStart)
        opacity = d.alpha * p
      } else {
        const pulse = Math.sin(t * 1.15 + d.phase) * 0.18 + 0.90
        opacity = d.alpha * pulse
      }

      ctx.beginPath()
      ctx.arc(d.x, d.y, d.r, 0, Math.PI * 2)
      ctx.fillStyle = `rgba(210, 82, 62, ${opacity})`
      ctx.fill()
    } else {
      const fadeStart = d.deathTime
      const fadeDur = 1.2

      let opacity = d.alpha

      if (t >= fadeStart && t < fadeStart + fadeDur) {
        const p = (t - fadeStart) / fadeDur
        opacity = d.alpha * (1 - p) + 0.035 * p
      } else if (t >= fadeStart + fadeDur) {
        opacity = 0.13
      }

      ctx.beginPath()
      ctx.arc(d.x, d.y, d.r, 0, Math.PI * 2)
      ctx.fillStyle = `rgba(95, 91, 85, ${opacity})`
      ctx.fill()
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
      circle at 32% 55%,
      rgba(250, 250, 248, 0.68) 0%,
      rgba(250, 250, 248, 0.50) 28%,
      rgba(250, 250, 248, 0.24) 58%,
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
  transform: translateY(-4vh);
}

.kicker {
  display: block;
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 12px;
  font-weight: 500;
  letter-spacing: 0.14em;
  text-transform: uppercase;
  color: #9d968e;
  margin-bottom: 34px;
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
  line-height: 1.45;
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
  color: #878078;
  font-weight: 400;
  margin-top: 8px;
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
    transform: translateY(-2vh);
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