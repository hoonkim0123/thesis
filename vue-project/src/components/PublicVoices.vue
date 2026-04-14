<script setup>
import { ref, onMounted } from 'vue'

const SUPPORT = [
  {
    label: 'Restaurant survival',
    count: 16,
    quote: '"With the high cost of rent and ingredients, expanded seating is a boon to restaurants."',
  },
  {
    label: 'Vibrant public space',
    count: 12,
    quote: '"It has given NYC a much more welcoming air — people laughing and talking where cars used to park."',
  },
  {
    label: 'Outdoor accessibility',
    count: 8,
    quote: '"Elderly and immunocompromised New Yorkers deserve safe ways to interact with what our city has to offer."',
  },
  {
    label: 'Pandemic recovery',
    count: 7,
    quote: '"During the pandemic, outdoor dining was vital for keeping these businesses open."',
  },
]

const OPPOSITION = [
  {
    label: 'Noise',
    count: 13,
    quote: '"Constant noise, crowds, and less livable streets for residents — a beer garden outside my window until midnight."',
  },
  {
    label: 'Sanitation',
    count: 10,
    quote: '"As soon as these structures were erected, rats were seen daily. It smelled like death in front of my office."',
  },
  {
    label: 'Sidewalk obstruction',
    count: 9,
    quote: '"Sidewalk blocked, bike lane conflicts, ADA compliance issues — the clearpath is simply not enough."',
  },
  {
    label: 'Visual clutter',
    count: 6,
    quote: '"The corrugated metal fences and umbrellas turned our historic neighborhood into an eyesore."',
  },
  {
    label: 'Safety concerns',
    count: 5,
    quote: '"Emergency vehicles cant navigate streets blocked by sidewalk dining structures this puts lives at risk."',
  },
]

const MAX_COUNT = 16
const activeQuote = ref(null)
const activeType = ref(null)

function hover(item, type) {
  activeQuote.value = item.quote
  activeType.value = type
}

function leave() {
  activeQuote.value = null
  activeType.value = null
}

// animate bars on mount
const mounted = ref(false)
onMounted(() => {
  setTimeout(() => { mounted.value = true }, 100)
})
</script>

<template>
  <div class="voices-wrap">

    <div class="voices-header">
      <div class="voices-label">Public Responses to Outdoor Dining</div>
      <div class="voices-description">
        <p>Outdoor dining was widely supported, but also contested. Public comments reveal both the value it created and the frictions it introduced.</p>
      </div>
      <div class="voices-note">
        <div>Source: NYC DOT Dining Out NYC rulemaking public comments · Comment period: Oct 20 – Nov 20, 2023</div>
        <div>Categories based on manual coding of sampled comments.</div>
        <div>Counts reflect coded mentions in a sample of public comments. Comments may include more than one theme.</div>
      </div>
    </div>

    <div class="voices-grid">

      <!-- Support -->
      <div class="voices-col">
        <div class="col-header col-header--support">
          <span class="col-title">In Support</span>
          <span class="col-desc">Why outdoor dining should exist</span>
        </div>

        <div class="bar-list">
          <div
            v-for="item in SUPPORT"
            :key="item.label"
            class="bar-row"
            @mouseenter="hover(item, 'support')"
            @mouseleave="leave"
          >
            <div class="bar-label">{{ item.label }}</div>
            <div class="bar-track">
              <div
                class="bar-fill bar-fill--support"
                :style="{ width: mounted ? (item.count / MAX_COUNT * 100) + '%' : '0%' }"
              ></div>
            </div>
            <div class="bar-count">{{ item.count }}</div>
          </div>
        </div>
      </div>

      <!-- Divider -->
      <div class="voices-divider">
        <div class="divider-line"></div>
      </div>

      <!-- Opposition -->
      <div class="voices-col">
        <div class="col-header col-header--oppose">
          <span class="col-title">In Opposition</span>
          <span class="col-desc">Why outdoor dining creates friction</span>
        </div>

        <div class="bar-list">
          <div
            v-for="item in OPPOSITION"
            :key="item.label"
            class="bar-row"
            @mouseenter="hover(item, 'oppose')"
            @mouseleave="leave"
          >
            <div class="bar-label">{{ item.label }}</div>
            <div class="bar-track">
              <div
                class="bar-fill bar-fill--oppose"
                :style="{ width: mounted ? (item.count / MAX_COUNT * 100) + '%' : '0%' }"
              ></div>
            </div>
            <div class="bar-count">{{ item.count }}</div>
          </div>
        </div>
      </div>

    </div>

    <!-- Quote panel -->
    <div class="quote-panel" :class="{ show: activeQuote, 'quote-panel--oppose': activeType === 'oppose' }">
      <div class="quote-mark">"</div>
      <div class="quote-text">{{ activeQuote }}</div>
      <div class="quote-src">Representative excerpt from NYC DOT rulemaking public comment, 2023</div>
    </div>

    <div class="voices-caption" v-if="!activeQuote">
      Hover a category to read a representative comment.
    </div>

  </div>
</template>

<style scoped>
.voices-wrap {
  width: 100%;
  margin-top: 40px;
}

.voices-header {
  margin-bottom: 28px;
}

.voices-label {
  font-family: "IBM Plex Sans", sans-serif;
  font-size: 16px;
  font-weight: 600;
  color: var(--ink);
  margin-bottom: 12px;
  letter-spacing: -0.01em;
}

.voices-description {
  font-family: "IBM Plex Sans", sans-serif;
  font-size: 14px;
  color: var(--ink);
  line-height: 1.6;
  margin-bottom: 16px;
  max-width: 600px;
}

.voices-description p {
  margin: 0;
}

.voices-note {
  font-family: "IBM Plex Mono", monospace;
  font-size: 10px;
  color: var(--ghost);
  line-height: 1.5;
  letter-spacing: 0.02em;
}

/* grid */
.voices-grid {
  display: grid;
  grid-template-columns: 1fr 1px 1fr;
  gap: 0 48px;
  align-items: start;
}

.voices-divider {
  display: flex;
  justify-content: center;
}

.divider-line {
  width: 1px;
  height: 100%;
  background: var(--rule);
  min-height: 200px;
}

/* column headers */
.col-header {
  display: flex;
  flex-direction: column;
  gap: 3px;
  padding-bottom: 12px;
  border-bottom: 2px solid var(--ink);
  margin-bottom: 4px;
}

.col-title {
  font-family: "IBM Plex Sans", sans-serif;
  font-size: 13px;
  font-weight: 600;
  color: var(--ink);
  letter-spacing: -0.01em;
}

.col-header--oppose .col-title {
  color: var(--ink);
}

.col-desc {
  font-family: "IBM Plex Mono", monospace;
  font-size: 10px;
  color: var(--ghost);
  letter-spacing: 0.02em;
}

/* bar rows */
.bar-list {
  display: flex;
  flex-direction: column;
}

.bar-row {
  display: grid;
  grid-template-columns: 140px 1fr 36px;
  align-items: center;
  gap: 10px;
  padding: 11px 6px;
  border-bottom: 1px solid var(--rule);
  cursor: default;
  transition: background 0.1s;
}

.bar-row:hover {
  background: var(--off);
}

.bar-label {
  font-family: "IBM Plex Sans", sans-serif;
  font-size: 13px;
  color: var(--muted);
  line-height: 1.3;
}

.bar-row:hover .bar-label {
  color: var(--ink);
}

.bar-track {
  height: 4px;
  background: var(--rule);
  border-radius: 2px;
  overflow: hidden;
}

.bar-fill {
  height: 100%;
  border-radius: 2px;
  transition: width 0.8s cubic-bezier(0.16, 1, 0.3, 1);
}

.bar-fill--support {
  background: var(--ink);
}

.bar-fill--oppose {
  background: var(--ink);
  opacity: 0.45;
}

.bar-pct {
  font-family: "IBM Plex Mono", monospace;
  font-size: 11px;
  color: var(--ghost);
  text-align: right;
}

.bar-count {
  font-family: "IBM Plex Mono", monospace;
  font-size: 11px;
  color: var(--ghost);
  text-align: right;
}

/* quote panel */
.quote-panel {
  margin-top: 12px;
  padding: 18px 20px;
  border-left: 2px solid var(--ink);
  background: var(--off);
  opacity: 0;
  transform: translateY(4px);
  transition: opacity 0.2s ease, transform 0.2s ease;
  pointer-events: none;
  min-height: 80px;
}

.quote-panel.show {
  opacity: 1;
  transform: translateY(0);
  pointer-events: auto;
}

.quote-panel--oppose {
  border-left-color: var(--ghost);
}

.quote-mark {
  font-family: "EB Garamond", serif;
  font-size: 32px;
  color: var(--ghost);
  line-height: 1;
  margin-bottom: 4px;
}

.quote-text {
  font-family: "IBM Plex Sans", sans-serif;
  font-size: 14px;
  color: var(--ink);
  line-height: 1.6;
  margin-bottom: 8px;
  font-style: italic;
}

.quote-src {
  font-family: "IBM Plex Mono", monospace;
  font-size: 10px;
  color: var(--ghost);
  letter-spacing: 0.04em;
}

.voices-caption {
  font-family: "IBM Plex Mono", monospace;
  font-size: 11px;
  color: var(--ghost);
  margin-top: 20px;
  letter-spacing: 0.03em;
}

@media (max-width: 760px) {
  .voices-grid {
    grid-template-columns: 1fr;
    gap: 32px 0;
  }
  .voices-divider { display: none; }
  .bar-row {
    grid-template-columns: 120px 1fr 32px;
  }
}
</style>
