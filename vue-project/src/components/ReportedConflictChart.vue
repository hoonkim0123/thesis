<script setup>
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'

const groups = [
  {
    id: 'movement',
    group: 'Blocked movement',
    description: 'Complaints about blocked sidewalk paths or street access.',
    items: [
      { label: 'Sidewalk blocked', value: 2754 },
      { label: 'Street blocked', value: 435 },
    ],
  },
  {
    id: 'setup',
    group: 'Physical setup',
    description: 'Complaints about structures, barriers, and installation conditions.',
    items: [
      { label: 'Setup condition', value: 2018 },
      { label: 'Barrier condition', value: 390 },
    ],
  },
  {
    id: 'status',
    group: 'Permit and status',
    description: 'Complaints about authorization, compliance, or restaurant status.',
    items: [
      { label: 'Unauthorized', value: 464 },
      { label: 'Restaurant status', value: 369 },
    ],
  },
]

const chartEl = ref(null)
const isVisible = ref(false)
let observer = null

const maxItemValue = computed(() => {
  return Math.max(...groups.flatMap((group) => group.items.map((item) => item.value)))
})

function getBarWidth(item) {
  if (!isVisible.value) return '0%'
  return `${(item.value / maxItemValue.value) * 100}%`
}

function getDelay(groupIndex, itemIndex) {
  if (!isVisible.value) return '0ms'
  return `${120 + groupIndex * 160 + itemIndex * 80}ms`
}

onMounted(() => {
  observer = new IntersectionObserver(
    ([entry]) => {
      if (entry.isIntersecting) {
        isVisible.value = true
      } else {
        isVisible.value = false
      }
    },
    {
      threshold: 0.25,
      rootMargin: '0px 0px -10% 0px',
    }
  )

  if (chartEl.value) {
    observer.observe(chartEl.value)
  }
})

onBeforeUnmount(() => {
  observer?.disconnect()
})
</script>

<template>
  <div ref="chartEl" class="rcc-wrap">
    <div class="rcc-kicker">311 complaint themes</div>

    <div class="rcc-grid">
      <article
        v-for="(group, groupIndex) in groups"
        :key="group.id"
        class="rcc-card"
      >
        <div class="rcc-card-top">
          <div class="rcc-card-title">{{ group.group }}</div>
          <p class="rcc-card-desc">{{ group.description }}</p>
        </div>

        <div class="rcc-rows">
          <div
            v-for="(item, itemIndex) in group.items"
            :key="item.label"
            class="rcc-row"
          >
            <div class="rcc-row-head">
              <span>{{ item.label }}</span>
              <strong>{{ item.value.toLocaleString() }}</strong>
            </div>

            <div class="rcc-track">
              <div
                class="rcc-fill"
                :class="{ 'rcc-fill-primary': item.label === 'Sidewalk blocked' }"
                :style="{
                  width: getBarWidth(item),
                  transitionDelay: getDelay(groupIndex, itemIndex),
                }"
              ></div>
            </div>
          </div>
        </div>
      </article>
    </div>

    <p class="rcc-note">
      Groups are interpretive categories based on coded 311 issue mentions. One complaint can include more than one theme.
    </p>
  </div>
</template>

<style scoped>
.rcc-wrap {
  width: 100%;
  margin: 32px 0 0;
}

.rcc-kicker {
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 10px;
  font-weight: 600;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: #aaa39c;
  margin-bottom: 18px;
}

.rcc-grid {
  display: grid;
  grid-template-columns: 1fr;
  border-top: 1px solid rgba(17, 17, 17, 0.10);
  border-bottom: 1px solid rgba(17, 17, 17, 0.10);
  background: transparent;
}

.rcc-card {
  display: grid;
  grid-template-columns: 230px 1fr;
  gap: 28px;
  padding: 24px 0;
  border-bottom: 1px solid rgba(17, 17, 17, 0.08);
}

.rcc-card:last-child {
  border-bottom: none;
}

.rcc-card-top {
  padding: 0;
  background: transparent;
  border-right: none;
}

.rcc-card-title {
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: 18px;
  line-height: 1.18;
  letter-spacing: -0.025em;
  font-weight: 700;
  color: var(--ink, #111111);
  margin-bottom: 8px;
}

.rcc-card-desc {
  margin: 0;
  max-width: 190px;
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: 13px;
  line-height: 1.5;
  color: #6e6862;
}

.rcc-rows {
  padding: 0;
  display: grid;
  gap: 16px;
}

.rcc-row {
  padding: 0;
  border-bottom: none;
}

.rcc-row-head {
  display: flex;
  justify-content: space-between;
  gap: 16px;
  margin-bottom: 7px;
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: 14px;
  line-height: 1.35;
  color: #5f5a55;
}

.rcc-row-head strong {
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 11px;
  font-weight: 700;
  color: #3e3a36;
}

.rcc-track {
  width: 100%;
  height: 5px;
  background: rgba(180, 74, 60, 0.16);
  overflow: hidden;
}

.rcc-fill {
  height: 100%;
  background: rgba(180, 74, 60, 0.52);
  transition-property: width;
  transition-duration: 900ms;
  transition-timing-function: cubic-bezier(0.16, 1, 0.3, 1);
}

.rcc-fill-primary {
  background: var(--accent, #b44a3c);
}

.rcc-note {
  margin-top: 12px;
  max-width: 760px;
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 9px;
  line-height: 1.5;
  color: #aaa39c;
}

@media (prefers-reduced-motion: reduce) {
  .rcc-fill {
    transition: none;
  }
}

@media (max-width: 760px) {
  .rcc-card {
    grid-template-columns: 1fr;
    gap: 16px;
  }

  .rcc-card-desc {
    max-width: none;
  }

  .rcc-row-head {
    font-size: 13px;
  }
}
</style>