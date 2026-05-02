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
    quote: '"It has given NYC a much more welcoming air. People laughing and talking where cars used to park."',
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
    quote: '"Sidewalk blocked, bike lane conflicts, ADA compliance issues. The clear path is simply not enough."',
  },
  {
    label: 'Noise',
    count: 167,
    quote: '"Constant noise, crowds, and less livable streets for residents. A beer garden outside my window until midnight."',
  },
  {
    label: 'Traffic safety',
    count: 150,
    quote: '"Emergency vehicles cannot navigate streets blocked by dining structures. This puts lives at risk."',
  },
  {
    label: 'Fees and fairness',
    count: 132,
    quote: '"Why are some businesses paying for sidewalk space while others are exempt? This system is fundamentally unfair."',
  },
]

const MAX_COUNT = 260

const activeQuote = ref(null)
const activeCategory = ref(null)
const activeLabel = ref(null)

function hover(item) {
  activeQuote.value = item.quote
  activeCategory.value = item.label.toUpperCase()
  activeLabel.value = item.label
}

function leave() {
  activeQuote.value = null
  activeCategory.value = null
  activeLabel.value = null
}

const mounted = ref(false)

onMounted(() => {
  setTimeout(() => {
    mounted.value = true
  }, 100)
})
</script>

<template>
  <div class="voices-wrap">
    <div class="quote-panel" :class="{ 'quote-panel--active': activeQuote }">
      <div class="quote-label">
        {{ activeCategory || 'REPRESENTATIVE COMMENT' }}
      </div>

      <div class="quote-text" :class="{ 'quote-text--idle': !activeQuote }">
        {{ activeQuote || 'Comments reveal two different ideas of public space.' }}
      </div>
    </div>

    <div class="voices-grid">
      <div class="voices-col">
        <div class="col-header">
          <span class="col-title">What people wanted to keep</span>
        </div>

        <div class="bar-list">
          <div
            v-for="item in SUPPORT"
            :key="item.label"
            class="bar-row"
            :class="{ 'is-active': activeLabel === item.label }"
            @mouseenter="hover(item)"
            @mouseleave="leave"
          >
            <div class="bar-label">{{ item.label }}</div>

            <div class="bar-track">
              <div
                class="bar-fill"
                :style="{ width: mounted ? (item.count / MAX_COUNT * 100) + '%' : '0%' }"
              ></div>
            </div>

            <div class="bar-count">{{ item.count }}</div>
          </div>
        </div>
      </div>

      <div class="voices-col">
        <div class="col-header">
          <span class="col-title">What people pushed back against</span>
        </div>

        <div class="bar-list">
          <div
            v-for="item in OPPOSITION"
            :key="item.label"
            class="bar-row"
            :class="{ 'is-active': activeLabel === item.label }"
            @mouseenter="hover(item)"
            @mouseleave="leave"
          >
            <div class="bar-label">{{ item.label }}</div>

            <div class="bar-track">
              <div
                class="bar-fill"
                :style="{ width: mounted ? (item.count / MAX_COUNT * 100) + '%' : '0%' }"
              ></div>
            </div>

            <div class="bar-count">{{ item.count }}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.voices-wrap {
  width: 100%;
  margin-top: 32px;
}

.quote-panel {
  margin: 0 0 28px;
  padding: 22px 26px;
  min-height: 92px;
  background: var(--off);
  border: 1px solid var(--rule);
  transition:
    background 0.15s ease,
    border-color 0.15s ease;
}

.quote-panel--active {
  background: var(--accent-pale);
  border-color: var(--accent-soft);
}

.quote-label {
  font-family: "IBM Plex Mono", monospace;
  font-size: 10px;
  font-weight: 600;
  letter-spacing: 0.11em;
  text-transform: uppercase;
  color: var(--ghost);
  margin-bottom: 9px;
  line-height: 1;
}

.quote-panel--active .quote-label {
  color: var(--accent);
}

.quote-text {
  font-family: "IBM Plex Sans", sans-serif;
  font-size: 15px;
  line-height: 1.55;
  color: var(--ink);
  font-style: italic;
  max-width: 860px;
}

.quote-text--idle {
  font-style: normal;
  color: var(--muted);
}

.voices-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 56px;
  align-items: start;
}

.col-header {
  padding-bottom: 12px;
  border-bottom: 2px solid var(--ink);
  margin-bottom: 4px;
}

.col-title {
  font-family: "IBM Plex Sans", sans-serif;
  font-size: 14px;
  font-weight: 700;
  color: var(--ink);
  letter-spacing: -0.01em;
}

.bar-list {
  display: flex;
  flex-direction: column;
}

.bar-row {
  display: grid;
  grid-template-columns: 150px 1fr 38px;
  align-items: center;
  gap: 12px;
  padding: 12px 8px;
  border-bottom: 1px solid var(--rule);
  cursor: default;
  transition: background 0.12s ease;
}

.bar-row:hover,
.bar-row.is-active {
  background: var(--accent-pale);
}

.bar-label {
  font-family: "IBM Plex Sans", sans-serif;
  font-size: 14px;
  color: var(--muted);
  line-height: 1.3;
  transition: color 0.12s ease;
}

.bar-count {
  font-family: "IBM Plex Mono", monospace;
  font-size: 12px;
  color: var(--muted);
  text-align: right;
  transition: color 0.12s ease;
}

.bar-track {
  height: 4px;
  background: var(--accent-soft);
  border-radius: 2px;
  overflow: hidden;
  transition: background 0.12s ease;
}

.bar-fill {
  height: 100%;
  border-radius: 2px;
  background: var(--accent);
  transition:
    width 0.8s cubic-bezier(0.16, 1, 0.3, 1),
    background 0.12s ease;
}

.bar-row:hover .bar-label,
.bar-row:hover .bar-count,
.bar-row.is-active .bar-label,
.bar-row.is-active .bar-count {
  color: var(--accent-dark);
}

.bar-row:hover .bar-track,
.bar-row.is-active .bar-track {
  background: #ead0ca;
}

.bar-row:hover .bar-fill,
.bar-row.is-active .bar-fill {
  background: var(--accent-dark);
}

@media (max-width: 760px) {
  .voices-grid {
    grid-template-columns: 1fr;
    gap: 40px;
  }

  .bar-row {
    grid-template-columns: 132px 1fr 34px;
  }

  .quote-panel {
    padding: 18px 20px;
    margin: 0 0 26px;
  }
}
</style>