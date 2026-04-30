<script setup>
const data = [
  { label: 'Blocked sidewalk', value: 2754 },
  { label: 'Setup condition', value: 2018 },
  { label: 'Unauthorized setup', value: 464 },
  { label: 'Blocked street', value: 435 },
  { label: 'Barrier condition', value: 390 },
  { label: 'Restaurant status', value: 369 },
]

const max = Math.max(...data.map(d => d.value))
</script>

<template>
  <div class="chart-wrap">
    <div class="chart-title">Reported outdoor dining issues</div>

    <div class="chart">
      <div v-for="d in data" :key="d.label" class="row">
        <span class="label">{{ d.label }}</span>
        <div class="bar">
          <div class="fill" :style="{ width: (d.value / max * 100) + '%' }"></div>
        </div>
        <span class="value">{{ d.value.toLocaleString() }}</span>
      </div>
    </div>

    <p class="chart-note">
      These complaints do not explain every disappearance. They show where outdoor dining became contested in street space.
    </p>
  </div>
</template>

<style scoped>
.chart-wrap {
  margin-top: 36px;
}

.chart-title {
  font-family: var(--mono);
  font-size: 11px;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: var(--ghost);
  margin-bottom: 18px;
}

.chart {
  width: 100%;
}

.row {
  display: grid;
  grid-template-columns: 200px 1fr 64px;
  align-items: center;
  column-gap: 20px;
  padding: 9px 0;
}

.label {
  font-size: 14px;
  color: var(--muted);
}

.bar {
  height: 6px;
  background: var(--rule);
  border-radius: 999px;
  overflow: hidden;
}

.fill {
  height: 100%;
  background: var(--ink);
  border-radius: 999px;
}

.value {
  font-family: var(--mono);
  font-size: 12px;
  color: var(--ghost);
  text-align: right;
}

.chart-note {
  margin-top: 18px;
  font-family: var(--mono);
  font-size: 11px;
  line-height: 1.55;
  color: var(--ghost);
}
</style>