<script setup>
import { computed, onMounted, ref } from 'vue'

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

const mounted = ref(false)

const maxItemValue = computed(() => {
  return Math.max(...groups.flatMap((group) => group.items.map((item) => item.value)))
})

onMounted(() => {
  setTimeout(() => {
    mounted.value = true
  }, 120)
})
</script>

<template>
  <div class="rcc-wrap">
    <div class="rcc-kicker">311 complaint themes</div>

    <div class="rcc-grid">
      <article
        v-for="group in groups"
        :key="group.id"
        class="rcc-card"
      >
        <div class="rcc-card-top">
          <div class="rcc-card-title">{{ group.group }}</div>
          <p class="rcc-card-desc">{{ group.description }}</p>
        </div>

        <div class="rcc-rows">
          <div
            v-for="item in group.items"
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
                :style="{ width: mounted ? (item.value / maxItemValue * 100) + '%' : '0%' }"
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
  font-size: 11px;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--ghost, #a0a0a0);
  margin-bottom: 14px;
}

.rcc-grid {
  display: grid;
  grid-template-columns: 1fr;
  border: 1px solid var(--rule, #e6e6e6);
  background: var(--white, #ffffff);
}

.rcc-card {
  display: grid;
  grid-template-columns: 250px 1fr;
  border-bottom: 1px solid var(--rule, #e6e6e6);
}

.rcc-card:last-child {
  border-bottom: none;
}

.rcc-card-top {
  padding: 22px 24px;
  background: var(--off, #f7f7f5);
  border-right: 1px solid var(--rule, #e6e6e6);
}

.rcc-card-title {
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: 19px;
  line-height: 1.18;
  letter-spacing: -0.025em;
  font-weight: 650;
  color: var(--ink, #111111);
  margin-bottom: 9px;
}

.rcc-card-desc {
  margin: 0;
  max-width: 220px;
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: 13px;
  line-height: 1.55;
  color: var(--muted, #555555);
}

.rcc-rows {
  padding: 16px 22px;
}

.rcc-row {
  padding: 11px 0 13px;
  border-bottom: 1px solid var(--rule, #e6e6e6);
}

.rcc-row:last-child {
  border-bottom: none;
}

.rcc-row-head {
  display: flex;
  justify-content: space-between;
  gap: 16px;
  margin-bottom: 9px;
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: 14px;
  line-height: 1.35;
  color: var(--muted, #555555);
}

.rcc-row-head strong {
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 12px;
  font-weight: 600;
  color: var(--ink, #111111);
}

.rcc-track {
  width: 100%;
  height: 6px;
  background: var(--accent-soft, #f0d8d3);
  overflow: hidden;
}

.rcc-fill {
  height: 100%;
  background: rgba(180, 74, 60, 0.5);
  transition: width 0.85s cubic-bezier(0.16, 1, 0.3, 1);
}

.rcc-fill-primary {
  background: var(--accent, #b44a3c);
}

.rcc-note {
  margin-top: 14px;
  max-width: 760px;
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 10px;
  line-height: 1.5;
  color: var(--ghost, #a0a0a0);
}

@media (max-width: 760px) {
  .rcc-lead {
    grid-template-columns: 1fr;
  }
}
</style>