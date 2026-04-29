<script setup>
const MEASURED = [
  'Outdoor dining nearby',
  'Restaurant / bar activity',
  'Reported issues nearby',
  'Street context'
]

const NOT_CAPTURED = [
  'Rules',
  'Operations',
  'Restaurant decisions'
]
</script>

<template>
  <div class="cs-wrap">
    <!-- Measured column -->
    <div class="cs-section">
      <div class="cs-header">Measured in this project</div>
      <div class="cs-list">
        <div v-for="item in MEASURED" :key="item" class="cs-item">
          {{ item }}
        </div>
      </div>
    </div>

    <!-- Also shaped by column -->
    <div class="cs-section">
      <div class="cs-header">Also shaped by</div>
      <div class="cs-list">
        <div v-for="item in NOT_CAPTURED" :key="item" class="cs-item">
          {{ item }}
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.cs-wrap {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 24px;
  margin: var(--space-5) 0;
}

.cs-section {
  padding: 16px 0;
}

.cs-header {
  font-family: var(--mono);
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: var(--ghost);
  margin-bottom: 12px;
  font-weight: 500;
}

.cs-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.cs-item {
  font-family: var(--sans);
  font-size: 13px;
  line-height: 1.4;
  color: var(--ink);
}

@media (max-width: 640px) {
  .cs-wrap {
    grid-template-columns: 1fr;
    gap: 16px;
  }
}
</style>
