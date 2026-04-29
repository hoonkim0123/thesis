<script setup>
import { ref, onMounted } from 'vue'

const SUPPORT = [
  {
    label: 'Year-round dining',
    count: 166,
    quote: '"Extended outdoor dining should be permanent, not just a pandemic emergency measure."',
  },
  {
    label: 'Flexible design',
    count: 158,
    quote: '"Better design standards and flexible structures would make outdoor dining work better for everyone."',
  },
  {
    label: 'Public life',
    count: 143,
    quote: '"It has given NYC a much more welcoming air — people laughing and talking where cars used to park."',
  },
  {
    label: 'Outdoor access',
    count: 123,
    quote: '"Elderly and immunocompromised New Yorkers deserve safe ways to interact with what our city has to offer."',
  },
  {
    label: 'Space for people',
    count: 52,
    quote: '"Reclaiming street space from cars and giving it back to people is exactly what the city needs."',
  },
]

const OPPOSITION = [
  {
    label: 'Sanitation / rats',
    count: 260,
    quote: '"As soon as these structures were erected, rats were seen daily. It smelled like death in front of my office."',
  },
  {
    label: 'Sidewalk obstruction',
    count: 208,
    quote: '"Sidewalk blocked, bike lane conflicts, ADA compliance issues — the clearpath is simply not enough."',
  },
  {
    label: 'Noise',
    count: 167,
    quote: '"Constant noise, crowds, and less livable streets for residents — a beer garden outside my window until midnight."',
  },
  {
    label: 'Traffic safety',
    count: 150,
    quote: '"Emergency vehicles cant navigate streets blocked by sidewalk dining structures this puts lives at risk."',
  },
  {
    label: 'Fees and fairness',
    count: 132,
    quote: '"Why are some businesses paying for sidewalk space while others are exempt? This system is fundamentally unfair."',
  },
]

const MAX_COUNT = 260
const activeQuote = ref('Hover a category to read a representative comment.')
const activeType = ref(null)
const activeCategory = ref(null)

function hover(item, type) {
  activeQuote.value = item.quote
  activeType.value = type
  activeCategory.value = item.label.toUpperCase()
}

function leave() {
  activeQuote.value = 'Hover a category to read a representative comment.'
  activeType.value = null
  activeCategory.value = null
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
        <p>Public comments reveal both the value outdoor dining created and the friction it introduced.</p>
      </div>
      <div class="voices-note">
        <div>Source: NYC DOT Dining Out NYC rulemaking public comments</div>
        <div>Categories based on keyword-assisted coding of 445 readable comments.</div>
        <div>Counts reflect coded theme mentions; one comment may include more than one theme.</div>
      </div>
    </div>

    <div class="voices-grid">

      <!-- Support -->
      <div class="voices-col">
        <div class="col-header col-header--support">
          <span class="col-title">Why outdoor dining should stay</span>
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
          <span class="col-title">Why outdoor dining created friction</span>
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
    <div class="quote-panel" :class="{ 'quote-panel--oppose': activeType === 'oppose' }">
      <div class="quote-category" v-if="activeType">{{ activeCategory }}</div>
      <div class="quote-mark" v-if="activeType">"</div>
      <div class="quote-text" :class="{ 'quote-text--placeholder': !activeType }">
        {{ activeQuote }}
      </div>
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
  display: none;
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
  display: none;
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
  transition: width 0.8s cubic-bezier(0.16, 1, 0.3, 1), background 0.15s;
  background: var(--data-historic);
}

.bar-fill--support {
  background: var(--data-historic);
}

.bar-fill--oppose {
  background: var(--data-historic);
}

.bar-row:hover .bar-fill--support,
.bar-row:hover .bar-fill--oppose {
  background: var(--accent);
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
  margin-top: 28px;
  padding: 18px 20px;
  background: var(--off);
  border: 1px solid var(--rule);
  min-height: 88px;
}

.quote-category {
  font-family: "IBM Plex Mono", monospace;
  font-size: 10px;
  font-weight: 500;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--ghost);
  margin-bottom: 8px;
  line-height: 1;
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

.quote-text--placeholder {
  font-family: "IBM Plex Mono", monospace;
  font-size: 13px;
  color: var(--muted);
  font-style: normal;
  letter-spacing: 0.03em;
  padding-top: 16px;
  text-align: left;
}

.quote-src {
  font-family: "IBM Plex Mono", monospace;
  font-size: 10px;
  color: var(--ghost);
  letter-spacing: 0.04em;
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
