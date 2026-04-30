<script setup>
import { computed, onMounted, ref } from 'vue'

const props = defineProps({
  activeCorridor: {
    type: String,
    default: 'Amsterdam Avenue',
  },
})

const rawFeatures = ref([])
const isLoading = ref(true)
const loadError = ref(false)

// ── Fixed: taller viewBox for a more substantial diagram ──
const SVG_W = 620
const SVG_H = 560

const CHART_LEFT = 72
const CHART_RIGHT = 580
const CHART_TOP = 60
const CHART_BOTTOM = 490
const CHART_H = CHART_BOTTOM - CHART_TOP

const CENTER_X = (CHART_LEFT + CHART_RIGHT) / 2
const ROADWAY_W = 132
const ROADWAY_X = CENTER_X - ROADWAY_W / 2

const SIDEWALK_W = 144
const LEFT_SIDEWALK_X = ROADWAY_X - SIDEWALK_W
const RIGHT_SIDEWALK_X = ROADWAY_X + ROADWAY_W

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

function getLicenseType(feature) {
  return String(feature?.properties?.license_type || 'Unknown')
}

function getDiningZone(type) {
  const value = String(type || '').toLowerCase()
  if (value.includes('sidewalk')) return 'sidewalk'
  if (value.includes('roadway')) return 'roadway'
  return 'unknown'
}

function getDotClass(type) {
  const zone = getDiningZone(type)
  if (zone === 'sidewalk') return 'cm-dot cm-dot-sidewalk'
  if (zone === 'roadway') return 'cm-dot cm-dot-roadway'
  return 'cm-dot cm-dot-unknown'
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
    const BASE = import.meta.env.BASE_URL
    const url = `${BASE}data/corridor_points_typed.geojson`
    const res = await fetch(url)
    if (!res.ok) throw new Error(`Could not load corridor_points_typed.geojson: ${res.status}`)
    const geojson = await res.json()
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
      label, street,
      p.corridor_label_clean, p.corridor_label,
      p.street_name, p.street_clean,
      p.full_street_name, p.corridor_name,
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

  const rawPoints = selectedFeatures.value
    .map((feature, index) => {
      const { lat, lng } = getLngLat(feature)
      const licenseType = getLicenseType(feature)
      return {
        id: index,
        lat, lng, feature, licenseType,
        zone: getDiningZone(licenseType),
      }
    })
    .filter((point) => Number.isFinite(point.lat) && Number.isFinite(point.lng))

  if (!rawPoints.length) return []

  const lats = rawPoints.map((p) => p.lat)
  const lngs = rawPoints.map((p) => p.lng)

  const minLatRaw = Math.min(...lats)
  const maxLatRaw = Math.max(...lats)
  const rawRange = maxLatRaw - minLatRaw || 0.0001
  const padding = rawRange * 0.22
  const minLat = minLatRaw - padding
  const maxLat = maxLatRaw + padding
  const latRange = maxLat - minLat || 0.0001

  const medianLng = [...lngs].sort((a, b) => a - b)[Math.floor(lngs.length / 2)]

  const mapped = rawPoints
    .map((point) => {
      const t = (maxLat - point.lat) / latRange
      const side = point.lng < medianLng ? -1 : 1
      return { ...point, side, y: CHART_TOP + t * CHART_H }
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
      rows.push({ y: point.y, items: [point] })
    }
  })

  const dots = []

  rows.forEach((row) => {
    const laneCount = {
      sidewalkLeft: 0, sidewalkRight: 0,
      roadwayLeft: 0, roadwayRight: 0,
      unknown: 0,
    }

    row.items.forEach((item) => {
      let baseOffset = 0
      let lane = 0

      if (item.zone === 'sidewalk') {
        if (item.side < 0) {
          lane = laneCount.sidewalkLeft++
          baseOffset = -128 - lane * 13
        } else {
          lane = laneCount.sidewalkRight++
          baseOffset = 128 + lane * 13
        }
      } else if (item.zone === 'roadway') {
        if (item.side < 0) {
          lane = laneCount.roadwayLeft++
          baseOffset = -38 - lane * 11
        } else {
          lane = laneCount.roadwayRight++
          baseOffset = 38 + lane * 11
        }
      } else {
        lane = laneCount.unknown++
        baseOffset = (lane % 2 === 0 ? -1 : 1) * (8 + lane * 6)
      }

      dots.push({
        id: item.id,
        x: CENTER_X + baseOffset,
        y: row.y,
        licenseType: item.licenseType,
      })
    })
  })

  return dots
})

const guideLines = computed(() => {
  const labels = activeMeta.value.guideLabels
  const positions = [0.1, 0.36, 0.64, 0.9]
  return labels.map((label, index) => ({
    label,
    y: CHART_TOP + positions[index] * CHART_H,
  }))
})
</script>

<template>
  <div class="cm-panel">
    <div v-if="isLoading" class="cm-state">Loading corridor diagram…</div>
    <div v-else-if="loadError" class="cm-state">Could not load corridor data.</div>

    <div v-else class="cm-inner">
      <!-- Corridor name header -->
      <div class="cm-header">
      </div>

      <svg
        class="cm-svg"
        :viewBox="`0 0 ${SVG_W} ${SVG_H}`"
        role="img"
        aria-label="Street section diagram showing roadway and sidewalk outdoor dining locations"
      >
        <!-- Guide lines -->
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
            :x="CHART_LEFT - 10"
            :y="guide.y + 4"
            text-anchor="end"
            class="cm-guide-label"
          >
            {{ guide.label }}
          </text>
        </g>

        <!-- Street cross-section -->
        <g>
          <rect
            :x="ROADWAY_X"
            :y="CHART_TOP - 18"
            :width="ROADWAY_W"
            :height="CHART_H + 36"
            rx="0"
            class="cm-roadway"
          />
          <line :x1="ROADWAY_X" :x2="ROADWAY_X"
            :y1="CHART_TOP - 10" :y2="CHART_BOTTOM + 10" class="cm-curb" />
          <line :x1="ROADWAY_X + ROADWAY_W" :x2="ROADWAY_X + ROADWAY_W"
            :y1="CHART_TOP - 10" :y2="CHART_BOTTOM + 10" class="cm-curb" />
          <line :x1="CENTER_X" :x2="CENTER_X"
            :y1="CHART_TOP - 10" :y2="CHART_BOTTOM + 10" class="cm-centerline" />
        </g>

        <!-- Zone labels -->
        <g>
          <text :x="LEFT_SIDEWALK_X + SIDEWALK_W / 2" :y="CHART_TOP - 30"
            text-anchor="middle" class="cm-zone-label">SIDEWALK</text>
          <text :x="CENTER_X" :y="CHART_TOP - 30"
            text-anchor="middle" class="cm-zone-label">ROADWAY</text>
          <text :x="RIGHT_SIDEWALK_X + SIDEWALK_W / 2" :y="CHART_TOP - 30"
            text-anchor="middle" class="cm-zone-label">SIDEWALK</text>
        </g>

        <!-- Dots -->
        <circle
          v-for="dot in stripDots"
          :key="dot.id"
          :cx="dot.x"
          :cy="dot.y"
          r="6"
          :class="getDotClass(dot.licenseType)"
        >
          <title>{{ dot.licenseType }}</title>
        </circle>
      </svg>
    </div>
  </div>
</template>

<style scoped>
.cm-panel {
  width: 100%;
  background: transparent;
}

.cm-inner {
  width: 100%;
}

/* Header above diagram */
.cm-header {
  display: flex;
  align-items: baseline;
  gap: 12px;
  margin-bottom: 16px;
  flex-wrap: wrap;
}

.cm-corridor-name {
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: 15px;
  font-weight: 600;
  color: var(--ink, #111);
}

.cm-corridor-area {
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 11px;
  color: var(--ghost, #aaa);
}

.cm-corridor-count {
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 11px;
  font-weight: 700;
  color: var(--ink, #111);
  margin-left: auto;
}

/* SVG fills its container — height determined by viewBox ratio */
.cm-svg {
  width: 100%;
  height: auto;
  display: block;
  background: transparent;
}

.cm-guide {
  stroke: var(--rule, #e6e6e6);
  stroke-width: 1;
  opacity: 0.7;
}

.cm-guide-label {
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 11px;
  fill: var(--ghost, #aaa);
}

.cm-roadway {
  fill: #f0ede9;
  opacity: 0.9;
}

.cm-curb {
  stroke: #d1cbc4;
  stroke-width: 1.2;
}

.cm-centerline {
  stroke: #beb7af;
  stroke-width: 1.2;
  stroke-dasharray: 6 7;
}

.cm-dot {
  opacity: 0.94;
  stroke: var(--white, #fff);
  stroke-width: 1.4;
}

.cm-dot-roadway  { fill: var(--ink, #111111); }
.cm-dot-sidewalk { fill: #6b665f; }
.cm-dot-unknown  { fill: var(--ghost, #aaa); opacity: 0.7; }

.cm-zone-label {
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 10px;
  letter-spacing: 0.08em;
  fill: var(--ghost, #aaa);
}

/* Legend */
.cm-legend {
  display: flex;
  gap: 20px;
  margin-top: 12px;
  flex-wrap: wrap;
}

.cm-leg-row {
  display: flex;
  align-items: center;
  gap: 6px;
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 10px;
  color: var(--muted, #666);
}

.cm-state {
  padding: 24px;
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 12px;
  color: var(--ghost, #aaa);
}
</style>