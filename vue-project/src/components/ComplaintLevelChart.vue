<script setup>
import { computed } from 'vue'

const props = defineProps({
  items: {
    type: Array,
    required: true,
  },
})

const maxValue = computed(() => {
  if (!props.items.length) return 1
  return Math.max(...props.items.map(d => d.value))
})
</script>

<template>
  <div class="complaint-card">
    <div class="chart-kicker">Complaint intensity across locations</div>

    <div v-for="item in items" :key="item.label" class="chart-row">
      <div class="row-label">{{ item.label }}</div>

      <div class="row-bar-wrap">
        <div class="row-track">
          <div
            class="row-fill"
            :class="item.className"
            :style="{ width: `${(item.value / maxValue) * 100}%` }"
          ></div>
        </div>
      </div>

      <div class="row-value">{{ item.value }}</div>
    </div>
  </div>
</template>

<style scoped>
.complaint-card {
  width: 100%;
  border-top: 1px solid #e6e6e6;
  padding-top: 18px;
}

.chart-kicker {
  font-size: 12px;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: #8f8f8f;
  margin-bottom: 18px;
}

.chart-row {
  display: grid;
  grid-template-columns: 180px 1fr 48px;
  align-items: center;
  gap: 16px;
  margin-bottom: 16px;
}

.row-label {
  font-size: 15px;
  color: #4d4d4d;
}

.row-bar-wrap {
  width: 100%;
}

.row-track {
  width: 100%;
  height: 12px;
  background: #e9e9e9;
  border-radius: 999px;
  overflow: hidden;
}

.row-fill {
  height: 100%;
  border-radius: 999px;
}

.row-fill.none {
  background: #cfcfcf;
}

.row-fill.low {
  background: #6bd48f;
}

.row-fill.moderate {
  background: #f4b942;
}

.row-fill.high {
  background: #e85c5c;
}

.row-value {
  font-size: 14px;
  color: #8b8b8b;
  text-align: right;
}
</style>
