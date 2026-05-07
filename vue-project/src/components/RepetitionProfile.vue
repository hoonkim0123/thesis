<template>
  <div class="repetition-profile">
    <div class="profile-axis">
      <span>More isolated</span>
      <div class="axis-line"></div>
      <span>More repeated</span>
    </div>

    <div class="profile-grid">
      <article
        v-for="profile in profiles"
        :key="profile.key"
        class="profile-card"
        :class="`profile-${profile.key}`"
      >
        <div class="card-top">
          <p class="card-kicker">{{ profile.label }}</p>
          <h3>{{ titleFor(profile.key) }}</h3>
        </div>

        <div class="street-scene">
          <div class="street-line"></div>

          <div
            v-for="block in setupBlocks(profile.key)"
            :key="block.id"
            class="setup-block"
            :class="{ active: profile.key !== 'isolated' }"
            :style="{
              left: `${block.left}%`,
              width: `${block.width}px`,
              opacity: blockOpacity(profile)
            }"
          ></div>

          <div
            v-for="dot in nearbyDots(profile.key)"
            :key="dot.id"
            class="nearby-dot"
            :style="{
              left: `${dot.left}%`,
              top: `${dot.top}%`,
              opacity: dotOpacity(profile)
            }"
          ></div>
        </div>

        <p class="card-caption">{{ profile.caption }}</p>

        <div class="signal-read">
          <span>model signal</span>
          <div class="signal-track">
            <div
              class="signal-fill"
              :style="{ width: `${signalWidth(profile)}%` }"
            ></div>
          </div>
        </div>
      </article>
    </div>

    <div class="profile-note">
      <span>Main signal</span>
      <strong>nearby same street repetition</strong>
      <strong>nearby outdoor dining</strong>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from 'vue'

const profiles = ref([
  {
    key: 'isolated',
    label: 'Isolated',
    caption: 'Little or no nearby same street repetition.',
    visual_strength: 0.18
  },
  {
    key: 'nearby',
    label: 'Nearby',
    caption: 'Some nearby same street repetition.',
    visual_strength: 0.48
  },
  {
    key: 'repeated',
    label: 'Repeated',
    caption: 'Several nearby setups repeat along the same street.',
    visual_strength: 1
  }
])

onMounted(async () => {
  const base = import.meta.env.BASE_URL || '/'

  try {
    const res = await fetch(`${base}data/s6_repetition_profile.json`)

    if (!res.ok) return

    const data = await res.json()

    profiles.value = data.map((item) => ({
      ...item,
      visual_strength: Math.max(0.16, item.visual_strength || 0)
    }))
  } catch (err) {
    console.warn('Using fallback S6 repetition profile:', err)
  }
})

function titleFor(key) {
  if (key === 'isolated') return 'Single point'
  if (key === 'nearby') return 'Nearby presence'
  return 'Repeated street presence'
}

function setupBlocks(key) {
  if (key === 'isolated') {
    return [
      { id: 'a', left: 50, width: 58 }
    ]
  }

  if (key === 'nearby') {
    return [
      { id: 'a', left: 38, width: 54 },
      { id: 'b', left: 62, width: 54 }
    ]
  }

  return [
    { id: 'a', left: 24, width: 50 },
    { id: 'b', left: 43, width: 50 },
    { id: 'c', left: 62, width: 50 },
    { id: 'd', left: 81, width: 50 }
  ]
}

function nearbyDots(key) {
  if (key === 'isolated') {
    return [
      { id: 'a', left: 20, top: 30 },
      { id: 'b', left: 76, top: 34 },
      { id: 'c', left: 30, top: 72 },
      { id: 'd', left: 84, top: 68 }
    ]
  }

  if (key === 'nearby') {
    return [
      { id: 'a', left: 26, top: 34 },
      { id: 'b', left: 40, top: 70 },
      { id: 'c', left: 60, top: 30 },
      { id: 'd', left: 74, top: 68 },
      { id: 'e', left: 50, top: 22 }
    ]
  }

  return [
    { id: 'a', left: 16, top: 32 },
    { id: 'b', left: 25, top: 70 },
    { id: 'c', left: 34, top: 26 },
    { id: 'd', left: 45, top: 72 },
    { id: 'e', left: 56, top: 28 },
    { id: 'f', left: 66, top: 70 },
    { id: 'g', left: 76, top: 30 },
    { id: 'h', left: 86, top: 66 }
  ]
}

function signalWidth(profile) {
  return Math.max(12, Math.min(100, profile.visual_strength * 100))
}

function blockOpacity(profile) {
  return 0.28 + profile.visual_strength * 0.62
}

function dotOpacity(profile) {
  return 0.18 + profile.visual_strength * 0.5
}
</script>

<style scoped>
.repetition-profile {
  margin-top: 46px;
  padding: 30px 0 28px;
  border-top: 1px solid var(--rule, rgba(0, 0, 0, 0.14));
  border-bottom: 1px solid var(--rule, rgba(0, 0, 0, 0.14));
}

.profile-axis {
  display: grid;
  grid-template-columns: 150px 1fr 150px;
  gap: 18px;
  align-items: center;
  margin-bottom: 26px;
}

.profile-axis span {
  font-family: var(--mono, monospace);
  font-size: 0.78rem;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: var(--ghost, #999);
}

.profile-axis span:last-child {
  text-align: right;
}

.axis-line {
  height: 1px;
  background: linear-gradient(
    to right,
    rgba(0, 0, 0, 0.08),
    rgba(180, 74, 60, 0.18),
    rgba(180, 74, 60, 0.48)
  );
}

.profile-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 16px;
}

.profile-card {
  min-height: 430px;
  display: flex;
  flex-direction: column;
  border: 1px solid rgba(0, 0, 0, 0.12);
  border-radius: 24px;
  padding: 22px;
  background: rgba(255, 255, 255, 0.5);
}

.profile-repeated {
  background: rgba(180, 74, 60, 0.045);
  border-color: rgba(180, 74, 60, 0.16);
}

.card-top {
  min-height: 116px;
}

.card-kicker {
  margin: 0 0 12px;
  font-family: var(--mono, monospace);
  font-size: 0.76rem;
  letter-spacing: 0.14em;
  text-transform: uppercase;
  color: var(--ghost, #999);
}

.card-top h3 {
  margin: 0;
  font-size: clamp(1.5rem, 2.1vw, 2.2rem);
  line-height: 1.05;
  letter-spacing: -0.03em;
  color: var(--ink, #111);
}

.street-scene {
  position: relative;
  height: 190px;
  margin: 6px 0 18px;
  border-radius: 20px;
  overflow: hidden;
  background: #f8f6f3;
  border: 1px solid rgba(0, 0, 0, 0.08);
}

.street-line {
  position: absolute;
  left: 9%;
  right: 9%;
  top: 50%;
  height: 18px;
  transform: translateY(-50%);
  border-radius: 999px;
  background: rgba(0, 0, 0, 0.055);
}

.setup-block {
  position: absolute;
  top: 50%;
  height: 48px;
  transform: translate(-50%, -50%);
  border-radius: 12px;
  background: #b44a3c;
  box-shadow: 0 0 0 7px rgba(180, 74, 60, 0.12);
}

.profile-isolated .setup-block {
  background: #aaa39c;
  box-shadow: 0 0 0 7px rgba(0, 0, 0, 0.07);
}

.nearby-dot {
  position: absolute;
  width: 9px;
  height: 9px;
  transform: translate(-50%, -50%);
  border-radius: 999px;
  background: #b44a3c;
}

.profile-isolated .nearby-dot {
  background: #aaa39c;
}

.card-caption {
  min-height: 58px;
  margin: 0 0 18px;
  font-size: 1rem;
  line-height: 1.42;
  color: var(--muted, #666);
}

.signal-read {
  margin-top: auto;
}

.signal-read span {
  display: block;
  margin-bottom: 9px;
  font-family: var(--mono, monospace);
  font-size: 0.72rem;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: var(--ghost, #999);
}

.signal-track {
  height: 8px;
  border-radius: 999px;
  background: rgba(0, 0, 0, 0.07);
  overflow: hidden;
}

.signal-fill {
  height: 100%;
  border-radius: 999px;
  background: #b44a3c;
}

.profile-note {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-top: 22px;
  align-items: center;
}

.profile-note span,
.profile-note strong {
  display: inline-flex;
  align-items: center;
  min-height: 32px;
  padding: 0 12px;
  border-radius: 999px;
  font-size: 0.82rem;
  font-weight: 500;
}

.profile-note span {
  color: var(--ghost, #8a8a8a);
  border: 1px solid rgba(0, 0, 0, 0.12);
}

.profile-note strong {
  color: #8e3329;
  background: rgba(180, 74, 60, 0.08);
  border: 1px solid rgba(180, 74, 60, 0.14);
}

@media (max-width: 980px) {
  .profile-grid {
    grid-template-columns: 1fr;
  }

  .profile-card {
    min-height: 0;
  }

  .card-top,
  .card-caption {
    min-height: 0;
  }
}

@media (max-width: 720px) {
  .profile-axis {
    grid-template-columns: 110px 1fr 110px;
  }

  .street-scene {
    height: 170px;
  }
}
</style>