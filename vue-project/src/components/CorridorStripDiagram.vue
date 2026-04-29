<script setup>
import { computed, onMounted, ref } from 'vue'

const props = defineProps({
  activeCorridor: {
    type: String,
    default: 'Amsterdam Avenue',
  },
})

const BASE = import.meta.env.BASE_URL

const rawFeatures = ref([])
const isLoading = ref(true)
const loadError = ref(false)

const SVG_W = 760
const SVG_H = 620

const CHART_LEFT = 120
const CHART_RIGHT = 560
const CHART_TOP = 86
const CHART_BOTTOM = 500
const CHART_H = CHART_BOTTOM - CHART_TOP

const STRIP_W = 76
const STRIP_X = CHART_LEFT + ((CHART_RIGHT - CHART_LEFT) / 2) - (STRIP_W / 2)

const LOCATOR_X = 620
const LOCATOR_Y = 468
const LOCATOR_W = 82
const LOCATOR_H = 82

const CORRIDORS = [
  {
    name: 'Amsterdam Avenue',
    area: 'Upper West Side',
    count: 19,
    keys: ['AMSTERDAM'],
    guideLabels: ['W 86th St', 'W 81st St', 'W 76th St', 'W 72nd St'],
  },
  {
    name: 'Columbus Avenue',
    area: 'Upper West Side',
    count: 13,
    keys: ['COLUMBUS'],
    guideLabels: ['W 86th St', 'W 81st St', 'W 76th St', 'W 72nd St'],
  },
  {
    name: '2nd Avenue',
    area: 'Upper East Side',
    count: 19,
    keys: ['2 AVENUE', '2ND AVENUE', 'SECOND AVENUE'],
    guideLabels: ['E 86th St', 'E 79th St', 'E 72nd St', 'E 66th St'],
  },
  {
    name: 'Mulberry Street',
    area: 'Little Italy / SoHo',
    count: 13,
    keys: ['MULBERRY'],
    guideLabels: ['Prince St', 'Spring St', 'Broome St', 'Canal St'],
  },
]

function canonicalize(value = '') {
  return String(value)
    .toLowerCase()
    .replace(/\./g, '')
    .replace(/\s+/g, ' ')
    .replace(/\bavenue\b/g, 'ave')
    .replace(/\bave\b/g, 'ave')
    .replace(/\bstreet\b/g, 'st')
    .replace(/\bst\b/g, 'st')
    .replace(/\bsecond\b/g, '2nd')
    .replace(/\b2 avenue\b/g, '2nd ave')
    .replace(/\b2 ave\b/g, '2nd ave')
    .trim()
}

function getFeatureLabel(feature) {
  return (
    feature?.properties?.corridor_label_clean ||
    feature?.properties?.corridor_label ||
    feature?.properties?.street_name ||
    feature?.properties?.street_clean ||
    feature?.properties?.full_street_name ||
    ''
  )
}

function getStreetFromLabel(label = '') {
  return String(label).split('(')[0].trim()
}

function getLngLat(feature) {
  const coords = feature?.geometry?.coordinates || []
  return {
    lng: Number(coords[0]),
    lat: Number(coords[1]),
  }
}

const activeMeta = computed(() => {
  const target = canonicalize(props.activeCorridor)

  return (
    CORRIDORS.find((corridor) => {
      const nameMatch = canonicalize(corridor.name) === target
      const keyMatch = corridor.keys.some((key) => target.includes(canonicalize(key)))
      return nameMatch || keyMatch
    }) || CORRIDORS[0]
  )
})

onMounted(async () => {
  try {
    const filesToTry = [
      'data/corridor_points.geojson',
      'data/corridor_points_labeled.geojson',
      'data/thesis_points_final_v2.geojson',
    ]

    let geojson = null

    for (const file of filesToTry) {
      const res = await fetch(`${BASE}${file}`)
      if (res.ok) {
        geojson = await res.json()
        break
      }
    }

    if (!geojson) {
      throw new Error('Could not load corridor point data')
    }

    rawFeatures.value = geojson.features || []
  } catch (error) {
    console.error('CorridorStripDiagram load error:', error)
    loadError.value = true
  } finally {
    isLoading.value = false
  }
})

const selectedFeatures = computed(() => {
  const meta = activeMeta.value

  const filtered = rawFeatures.value.filter((feature) => {
    const label = getFeatureLabel(feature)
    const street = getStreetFromLabel(label)

    const p = feature.properties || {}
    const candidates = [
      label,
      street,
      p.corridor_label_clean,
      p.corridor_label,
      p.street_name,
      p.street_clean,
      p.full_street_name,
      p.corridor_name,
    ].filter(Boolean)

    return candidates.some((candidate) => {
      const candidateText = canonicalize(candidate)
      return meta.keys.some((key) => candidateText.includes(canonicalize(key)))
    })
  })

  return filtered.slice(0, meta.count)
})

const stripDots = computed(() => {
  if (!selectedFeatures.value.length) return []

  const points = selectedFeatures.value
    .map((feature, index) => {
      const { lat, lng } = getLngLat(feature)
      return { id: index, lat, lng }
    })
    .filter((point) => Number.isFinite(point.lat) && Number.isFinite(point.lng))

  if (!points.length) return []

  const lats = points.map((point) => point.lat)
  const minLatRaw = Math.min(...lats)
  const maxLatRaw = Math.max(...lats)
  const rawRange = maxLatRaw - minLatRaw || 0.0001

  const padding = rawRange * 0.18
  const minLat = minLatRaw - padding
  const maxLat = maxLatRaw + padding
  const latRange = maxLat - minLat || 0.0001

  const mapped = points
    .map((point) => {
      const t = (maxLat - point.lat) / latRange
      return {
        ...point,
        y: CHART_TOP + t * CHART_H,
      }
    })
    .sort((a, b) => a.y - b.y)

  const rows = []
  const threshold = 18

  mapped.forEach((point) => {
    const last = rows[rows.length - 1]

    if (last && Math.abs(last.y - point.y) < threshold) {
      last.items.push(point)
      last.y = (last.y * (last.items.length - 1) + point.y) / last.items.length
    } else {
      rows.push({
        y: point.y,
        items: [point],
      })
    }
  })

  const offsets = [0, -14, 14, -28, 28, -42, 42, -56, 56]
  const dots = []

  rows.forEach((row) => {
    row.items.forEach((item, index) => {
      const offset = offsets[index] ?? ((index % 2 === 0 ? 1 : -1) * (56 + index * 4))

      dots.push({
        id: item.id,
        x: STRIP_X + STRIP_W / 2 + offset,
        y: row.y,
      })
    })
  })

  return dots
})

const guideLines = computed(() => {
  const labels = activeMeta.value.guideLabels
  const positions = [0.12, 0.38, 0.64, 0.87]

  return labels.map((label, index) => ({
    label,
    y: CHART_TOP + positions[index] * CHART_H,
  }))
})

const allLocatorDots = computed(() => {
  if (!rawFeatures.value.length) return []

  const coords = rawFeatures.value
    .map(getLngLat)
    .filter((point) => Number.isFinite(point.lat) && Number.isFinite(point.lng))

  if (!coords.length) return []

  const lats = coords.map((point) => point.lat)
  const lngs = coords.map((point) => point.lng)

  const minLat = Math.min(...lats)
  const maxLat = Math.max(...lats)
  const minLng = Math.min(...lngs)
  const maxLng = Math.max(...lngs)

  const latRange = maxLat - minLat || 0.0001
  const lngRange = maxLng - minLng || 0.0001

  return coords.map((point, index) => ({
    id: index,
    x: LOCATOR_X + ((point.lng - minLng) / lngRange) * LOCATOR_W,
    y: LOCATOR_Y + ((maxLat - point.lat) / latRange) * LOCATOR_H,
  }))
})

const selectedLocatorDots = computed(() => {
  if (!selectedFeatures.value.length || !rawFeatures.value.length) return []

  const allCoords = rawFeatures.value
    .map(getLngLat)
    .filter((point) => Number.isFinite(point.lat) && Number.isFinite(point.lng))

  const selectedCoords = selectedFeatures.value
    .map(getLngLat)
    .filter((point) => Number.isFinite(point.lat) && Number.isFinite(point.lng))

  if (!allCoords.length || !selectedCoords.length) return []

  const lats = allCoords.map((point) => point.lat)
  const lngs = allCoords.map((point) => point.lng)

  const minLat = Math.min(...lats)
  const maxLat = Math.max(...lats)
  const minLng = Math.min(...lngs)
  const maxLng = Math.max(...lngs)

  const latRange = maxLat - minLat || 0.0001
  const lngRange = maxLng - minLng || 0.0001

  return selectedCoords.map((point, index) => ({
    id: index,
    x: LOCATOR_X + ((point.lng - minLng) / lngRange) * LOCATOR_W,
    y: LOCATOR_Y + ((maxLat - point.lat) / latRange) * LOCATOR_H,
  }))
})
</script>

<template>
  <div class="cm-panel">
    <div v-if="isLoading" class="cm-state">Loading corridor diagram…</div>

    <div v-else-if="loadError" class="cm-state">Could not load corridor data.</div>

    <div v-else class="cm-inner">
      <div class="cm-meta">
        <div>
          <div class="cm-street">{{ activeMeta.name }}</div>
          <div class="cm-neighborhood">{{ activeMeta.area }}</div>
        </div>

        <div class="cm-count">{{ activeMeta.count }} locations</div>
      </div>

      <svg
        class="cm-svg"
        :viewBox="`0 0 ${SVG_W} ${SVG_H}`"
        role="img"
        aria-label="Street strip diagram showing repeated outdoor dining locations along the selected corridor"
      >
        <g>
          <line
            v-for="guide in guideLines"
            :key="guide.label"
            :x1="CHART_LEFT"
            :x2="CHART_RIGHT"
            :y1="guide.y"
            :y2="guide.y"
            class="cm-guide"
          />

          <text
            v-for="guide in guideLines"
            :key="'label-' + guide.label"
            :x="CHART_LEFT - 18"
            :y="guide.y + 4"
            text-anchor="end"
            class="cm-guide-label"
          >
            {{ guide.label }}
          </text>
        </g>

        <g>
          <rect
            :x="STRIP_X"
            :y="CHART_TOP - 18"
            :width="STRIP_W"
            :height="CHART_H + 36"
            rx="10"
            class="cm-roadway"
          />

          <line
            :x1="STRIP_X + 9"
            :x2="STRIP_X + 9"
            :y1="CHART_TOP - 10"
            :y2="CHART_BOTTOM + 10"
            class="cm-curb"
          />

          <line
            :x1="STRIP_X + STRIP_W - 9"
            :x2="STRIP_X + STRIP_W - 9"
            :y1="CHART_TOP - 10"
            :y2="CHART_BOTTOM + 10"
            class="cm-curb"
          />

          <line
            :x1="STRIP_X + STRIP_W / 2"
            :x2="STRIP_X + STRIP_W / 2"
            :y1="CHART_TOP - 10"
            :y2="CHART_BOTTOM + 10"
            class="cm-centerline"
          />
        </g>

        <circle
          v-for="dot in stripDots"
          :key="dot.id"
          :cx="dot.x"
          :cy="dot.y"
          r="5.6"
          class="cm-dot"
        />

        <text
          :x="(CHART_LEFT + CHART_RIGHT) / 2"
          :y="CHART_BOTTOM + 34"
          text-anchor="middle"
          class="cm-note"
        >
          Each dot marks one current location.
        </text>

        <g>
          <rect
            :x="LOCATOR_X - 10"
            :y="LOCATOR_Y - 10"
            :width="LOCATOR_W + 20"
            :height="LOCATOR_H + 20"
            class="cm-locator-box"
          />

          <circle
            v-for="dot in allLocatorDots"
            :key="'all-' + dot.id"
            :cx="dot.x"
            :cy="dot.y"
            r="1.7"
            class="cm-locator-dot-all"
          />

          <circle
            v-for="dot in selectedLocatorDots"
            :key="'selected-' + dot.id"
            :cx="dot.x"
            :cy="dot.y"
            r="3.2"
            class="cm-locator-dot-selected"
          />
        </g>
      </svg>
    </div>
  </div>
</template>

<style scoped>
.cm-panel {
  width: 100%;
  min-height: 620px;
  border: 1px solid var(--rule);
  background: var(--white);
}

.cm-inner {
  width: 100%;
}

.cm-meta {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  padding: 24px 28px 12px;
}

.cm-street {
  font-family: var(--sans);
  font-size: 28px;
  font-weight: 700;
  line-height: 1.08;
  color: var(--ink);
  margin-bottom: 6px;
}

.cm-neighborhood {
  font-family: var(--mono);
  font-size: 12px;
  letter-spacing: 0.03em;
  color: var(--ghost);
}

.cm-count {
  font-family: var(--mono);
  font-size: 14px;
  color: var(--ink);
  white-space: nowrap;
  padding-top: 6px;
}

.cm-svg {
  width: 100%;
  height: auto;
  display: block;
}

.cm-guide {
  stroke: var(--rule);
  stroke-width: 1;
}

.cm-guide-label {
  font-family: var(--mono);
  font-size: 11px;
  fill: var(--ghost);
}

.cm-roadway {
  fill: #f1efec;
}

.cm-curb {
  stroke: #d6d1cb;
  stroke-width: 1.1;
}

.cm-centerline {
  stroke: #c2bcb5;
  stroke-width: 1.2;
  stroke-dasharray: 6 6;
}

.cm-dot {
  fill: var(--ink);
  opacity: 0.92;
}

.cm-locator-box {
  fill: transparent;
  stroke: var(--rule);
  stroke-width: 1;
}

.cm-locator-dot-all {
  fill: #d0cbc5;
  opacity: 0.85;
}

.cm-locator-dot-selected {
  fill: var(--ink);
}

.cm-note {
  font-family: var(--mono);
  font-size: 11px;
  fill: var(--ghost);
}

.cm-state {
  padding: 24px;
  font-family: var(--mono);
  font-size: 12px;
  color: var(--ghost);
}
</style>