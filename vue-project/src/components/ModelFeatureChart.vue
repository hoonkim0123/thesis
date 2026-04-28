<script setup>
import { onMounted, ref } from 'vue'

const features = [
  { label: 'Cluster score',      side: 'surv', weight: 0.88, tag: 'Continuity' },
  { label: 'Liquor density',     side: 'surv', weight: 0.58, tag: 'Activity'   },
  { label: 'Complaint count',    side: 'loss', weight: 0.71, tag: 'Conflict'   },
  { label: 'Street width',       side: 'neut', weight: 0.22, tag: 'Space'      },
]

const MAX_PCT = 42 // max bar width as % from center

const chartRef = ref(null)
const animated = ref(false)

onMounted(() => {
  const observer = new IntersectionObserver(
    (entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting && !animated.value) {
          animated.value = true
        }
      })
    },
    { threshold: 0.2 }
  )
  if (chartRef.value) observer.observe(chartRef.value)
})

function barWidth(weight) {
  return animated.value ? (weight * MAX_PCT).toFixed(1) + '%' : '0%'
}
</script>

<template>
  <div class="mfc-wrap" ref="chartRef">

    <!-- Legend -->
    <div class="mfc-legend">
      <div class="mfc-legend-item">
        <span class="mfc-swatch sw-surv"></span>
        <span>Associated with survival</span>
      </div>
      <div class="mfc-legend-item">
        <span class="mfc-swatch sw-loss"></span>
        <span>Associated with disappearance</span>
      </div>
      <div class="mfc-legend-item">
        <span class="mfc-swatch sw-neut"></span>
        <span>Ambiguous</span>
      </div>
    </div>

    <!-- Chart rows -->
    <div class="mfc-chart">
      <template v-for="(f, i) in features" :key="f.label">

        <!-- Divider between survival and loss groups -->
        <div v-if="i === 2" class="mfc-divider"></div>

        <div class="mfc-row">
          <div class="mfc-label">{{ f.label }}</div>
          <div class="mfc-bars">
            <div class="mfc-axis"></div>

            <!-- Survival bar (extends right) -->
            <div
              v-if="f.side === 'surv'"
              class="mfc-bar mfc-bar-surv"
              :style="{ width: barWidth(f.weight) }"
            ></div>

            <!-- Loss bar (extends left) -->
            <div
              v-else-if="f.side === 'loss'"
              class="mfc-bar mfc-bar-loss"
              :style="{ width: barWidth(f.weight) }"
            ></div>

            <!-- Neutral bar (short, right, gray) -->
            <div
              v-else
              class="mfc-bar mfc-bar-neut"
              :style="{ width: barWidth(f.weight) }"
            ></div>

            <span
              class="mfc-tag"
              :class="f.side === 'loss' ? 'mfc-tag-loss' : 'mfc-tag-surv'"
              :style="{ '--tw': (f.weight * MAX_PCT).toFixed(1) + '%' }"
            >{{ f.tag }}</span>
          </div>
        </div>

      </template>
    </div>

    <!-- Axis labels -->
    <div class="mfc-axis-labels">
      <span>← Loss</span>
      <span>Survival →</span>
    </div>

  </div>
</template>

<style scoped>
.mfc-wrap {
  width: 100%;
  padding: 8px 0 4px;
}

/* Legend */
.mfc-legend {
  display: flex;
  gap: 20px;
  flex-wrap: wrap;
  margin-bottom: 20px;
}
.mfc-legend-item {
  display: flex;
  align-items: center;
  gap: 7px;
  font-family: var(--sans);
  font-size: 12px;
  color: var(--muted);
}
.mfc-swatch {
  width: 18px;
  height: 3px;
  border-radius: 2px;
  flex-shrink: 0;
}
.sw-surv { background: var(--ink); }
.sw-loss { background: var(--accent); }
.sw-neut { background: var(--ghost); }

/* Chart */
.mfc-chart {
  width: 100%;
}

.mfc-row {
  display: flex;
  align-items: center;
  margin-bottom: 3px;
}

.mfc-label {
  width: 160px;
  flex-shrink: 0;
  font-family: var(--sans);
  font-size: 12px;
  color: var(--muted);
  text-align: right;
  padding-right: 12px;
  line-height: 1.3;
}

.mfc-bars {
  flex: 1;
  position: relative;
  display: flex;
  align-items: center;
  height: 34px;
}

.mfc-axis {
  position: absolute;
  left: 50%;
  top: 0;
  bottom: 0;
  width: 1px;
  background: var(--rule);
  z-index: 1;
}

/* Bars */
.mfc-bar {
  position: absolute;
  height: 12px;
  top: 50%;
  transform: translateY(-50%);
  transition: width 0.8s cubic-bezier(0.22, 1, 0.36, 1);
}

.mfc-bar-surv {
  left: 50%;
  background: var(--ink);
  border-radius: 0 2px 2px 0;
}
.mfc-bar-loss {
  right: 50%;
  background: var(--accent);
  border-radius: 2px 0 0 2px;
}
.mfc-bar-neut {
  left: 50%;
  background: var(--ghost);
  border-radius: 0 2px 2px 0;
}

/* Tags */
.mfc-tag {
  position: absolute;
  font-family: var(--mono);
  font-size: 10px;
  letter-spacing: 0.07em;
  text-transform: uppercase;
  color: var(--ghost);
  white-space: nowrap;
  top: 50%;
  transform: translateY(-50%);
}
.mfc-tag-surv {
  left: calc(50% + var(--tw) + 8px);
}
.mfc-tag-loss {
  right: calc(50% + var(--tw) + 8px);
}

/* Divider between groups */
.mfc-divider {
  height: 1px;
  background: var(--rule);
  margin: 8px 0 8px 160px;
}

/* Axis labels */
.mfc-axis-labels {
  display: flex;
  justify-content: space-between;
  padding-left: 160px;
  margin-top: 6px;
  font-family: var(--sans);
  font-size: 11px;
  color: var(--ghost);
}

/* Responsive */
@media (max-width: 600px) {
  .mfc-label {
    width: 120px;
    font-size: 11px;
  }
}
</style>
