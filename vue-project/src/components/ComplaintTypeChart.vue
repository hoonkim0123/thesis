<script setup>
const data = [
  { label: 'Sidewalk blocked', value: 2754 },
  { label: 'Setup condition', value: 2018 },
  { label: 'Unauthorized', value: 464 },
  { label: 'Street blocked', value: 435 },
  { label: 'Barrier condition', value: 390 },
  { label: 'Restaurant status', value: 369 },
]

const max = Math.max(...data.map(d => d.value))
</script>

<template>
  <div class="chart">
    <div v-for="d in data" :key="d.label" class="row">
      <span class="label">{{ d.label }}</span>
      <div class="bar">
        <div class="fill" :style="{ width: (d.value / max * 100) + '%' }"></div>
      </div>
      <span class="value">{{ d.value }}</span>
    </div>
  </div>
</template>

<style scoped>
.chart {
  margin-top: 32px;
}

.row {
  display: grid;
  grid-template-columns: 200px 1fr 60px;
  align-items: center;
  margin-bottom: 12px;
}

.label {
  font-size: 14px;
  color: var(--muted);
}

.bar {
  height: 6px;
  background: var(--rule);
  border-radius: 3px;
}

.fill {
  height: 100%;
  background: var(--ink);
  border-radius: 3px;
}

.value {
  font-size: 12px;
  color: var(--ghost);
  text-align: right;
}
</style>