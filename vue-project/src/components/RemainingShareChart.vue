<script setup>
// All numbers derived from raw data pipeline
// Historic Manhattan (coord-deduped): 4,659
// Current licensed Manhattan: 318
// Corridor streets (Amsterdam, Columbus, Mulberry, 2 Ave): hist=343, curr=45
// Other streets: hist=4,316, curr=273

const BARS = [
  {
    label: 'All Manhattan locations',
    sublabel: '318 of 4,660',
    value: 6.8,
    highlight: false,
  },
  {
    label: 'Repeated corridor streets',
    sublabel: '45 of 343',
    value: 13.1,
    highlight: true,
  },
  {
    label: 'Other streets',
    sublabel: '273 of 4,316',
    value: 6.3,
    highlight: false,
  },
]

const MAX = 20 // x-axis max %
</script>

<template>
  <div class="rs-wrap">
    <div class="rs-title">Remaining share by street pattern</div>

    <div class="rs-chart">
      <!-- X axis labels -->
      <div class="rs-axis">
        <span>0%</span>
        <span>5%</span>
        <span>10%</span>
        <span>15%</span>
        <span>20%</span>
      </div>

      <!-- Grid lines -->
      <div class="rs-grid">
        <div class="rs-gridline" v-for="n in [0,5,10,15,20]" :key="n"
          :style="{ left: (n / MAX * 100) + '%' }" />
      </div>

      <!-- Bars -->
      <div class="rs-bars">
        <div
          v-for="b in BARS"
          :key="b.label"
          class="rs-row"
          :class="{ 'is-highlight': b.highlight }"
        >
          <div class="rs-label-col">
            <div class="rs-label">{{ b.label }}</div>
            <div class="rs-sublabel">{{ b.sublabel }}</div>
          </div>
          <div class="rs-bar-col">
            <div class="rs-bar-track">
              <div
                class="rs-bar-fill"
                :style="{ width: (b.value / MAX * 100) + '%' }"
              ></div>
              <span class="rs-bar-val" :style="{ left: (b.value / MAX * 100) + '%' }">{{ b.value }}%</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="rs-caption">
      Repeated corridor streets retained about twice the Manhattan average share of outdoor dining.
    </div>
  </div>
</template>

<style scoped>
.rs-wrap {
  margin: 36px 0 28px;
}

.rs-title {
  font-family: var(--mono);
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: var(--ghost);
  margin-bottom: 20px;
}

.rs-chart {
  position: relative;
  padding-bottom: 24px;
}

/* Grid */
.rs-grid {
  position: absolute;
  inset: 0 0 24px 200px;
  pointer-events: none;
}

.rs-gridline {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 1px;
  background: var(--rule);
}

/* Axis */
.rs-axis {
  display: flex;
  justify-content: space-between;
  margin-left: 200px;
  margin-bottom: 6px;
  font-family: var(--mono);
  font-size: 10px;
  color: var(--ghost);
}

/* Bars */
.rs-bars {
  display: flex;
  flex-direction: column;
  gap: 0;
}

.rs-row {
  display: grid;
  grid-template-columns: 200px 1fr;
  align-items: center;
  padding: 10px 0;
  border-top: 1px solid var(--rule);
  transition: background 0.15s;
}

.rs-row:last-child {
  border-bottom: 1px solid var(--rule);
}

.rs-row.is-highlight {
  background: transparent;
}

.rs-label-col {
  padding-right: 20px;
}

.rs-label {
  font-family: var(--sans);
  font-size: 13px;
  font-weight: 500;
  color: var(--ink);
  line-height: 1.3;
  margin-bottom: 2px;
}

.rs-row.is-highlight .rs-label {
  font-weight: 700;
}

.rs-sublabel {
  font-family: var(--mono);
  font-size: 10px;
  color: var(--ghost);
}

.rs-bar-col {
  padding-right: 16px;
}

.rs-bar-track {
  position: relative;
  height: 28px;
  display: flex;
  align-items: center;
}

.rs-bar-fill {
  height: 100%;
  background: var(--ink);
  opacity: 0.15;
  transition: width 0.6s ease;
}

.rs-row.is-highlight .rs-bar-fill {
  opacity: 1;
}

.rs-bar-val {
  position: absolute;
  transform: translateX(8px);
  font-family: var(--mono);
  font-size: 12px;
  font-weight: 700;
  color: var(--ink);
  white-space: nowrap;
}

.rs-row.is-highlight .rs-bar-val {
  font-size: 13px;
}

.rs-caption {
  margin-top: 12px;
  font-family: var(--mono);
  font-size: 11px;
  line-height: 1.5;
  color: var(--ghost);
}
</style>
