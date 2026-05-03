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

const SVG_W = 620
const SVG_H = 600

const CHART_LEFT = 58
const CHART_RIGHT = 608
const CHART_TOP = 95
const CHART_BOTTOM = 525
const CHART_H = CHART_BOTTOM - CHART_TOP

const CENTER_X = (CHART_LEFT + CHART_RIGHT) / 2

const STREET_W = 240
const STREET_X = CENTER_X - STREET_W / 2

// narrower sidewalks, wider parking, slimmer travel lanes
const SIDEWALK_W = 92
const PARKING_W = 56
const TRAVEL_W = 64

const LEFT_SIDEWALK_X = STREET_X - SIDEWALK_W
const LEFT_PARKING_X = STREET_X
const LEFT_TRAVEL_X = LEFT_PARKING_X + PARKING_W
const RIGHT_TRAVEL_X = LEFT_TRAVEL_X + TRAVEL_W
const RIGHT_PARKING_X = RIGHT_TRAVEL_X + TRAVEL_W
const RIGHT_SIDEWALK_X = RIGHT_PARKING_X + PARKING_W

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

    if (!res.ok) {
      throw new Error(`Could not load corridor_points_typed.geojson: ${res.status}`)
    }

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

  const rawPoints = selectedFeatures.value
    .map((feature, index) => {
      const { lat, lng } = getLngLat(feature)
      const licenseType = getLicenseType(feature)

      return {
        id: `${activeMeta.value.name}-${index}`,
        lat,
        lng,
        feature,
        licenseType,
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

      return {
        ...point,
        side,
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
      rows.push({ y: point.y, items: [point] })
    }
  })

  const dots = []

  rows.forEach((row) => {
    const laneCount = {
      sidewalkLeft: 0,
      sidewalkRight: 0,
      roadwayLeft: 0,
      roadwayRight: 0,
      unknown: 0,
    }

    row.items.forEach((item) => {
      let baseOffset = 0
      let lane = 0

      if (item.zone === 'sidewalk') {
        if (item.side < 0) {
          lane = laneCount.sidewalkLeft++
          baseOffset = -146 - lane * 10
        } else {
          lane = laneCount.sidewalkRight++
          baseOffset = 146 + lane * 10
        }
      } else if (item.zone === 'roadway') {
        // Roadway dining is shown as curbside / parking-lane space,
        // not as occupying the middle travel lanes.
        if (item.side < 0) {
          lane = laneCount.roadwayLeft++
          baseOffset = -78 - lane * 9
        } else {
          lane = laneCount.roadwayRight++
          baseOffset = 78 + lane * 9
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
      <svg
        class="cm-svg"
        :viewBox="`0 0 ${SVG_W} ${SVG_H}`"
        role="img"
        aria-label="Street section diagram showing sidewalk, parking lane, travel lane, and outdoor dining locations"
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
            :x="CHART_LEFT - 8"
            :y="guide.y + 4"
            text-anchor="start"
            class="cm-guide-label"
          >
            {{ guide.label }}
          </text>
        </g>

        <!-- Street cross-section -->
        <g>
          <!-- Full street allocation -->
          <rect
            :x="STREET_X"
            :y="CHART_TOP - 12"
            :width="STREET_W"
            :height="CHART_H + 24"
            class="cm-street-base"
          />

          <!-- Parking lanes -->
          <rect
            :x="LEFT_PARKING_X"
            :y="CHART_TOP - 12"
            :width="PARKING_W"
            :height="CHART_H + 24"
            class="cm-parking"
          />

          <rect
            :x="RIGHT_PARKING_X"
            :y="CHART_TOP - 12"
            :width="PARKING_W"
            :height="CHART_H + 24"
            class="cm-parking"
          />

          <!-- Travel lanes -->
          <rect
            :x="LEFT_TRAVEL_X"
            :y="CHART_TOP - 12"
            :width="TRAVEL_W"
            :height="CHART_H + 24"
            class="cm-travel"
          />

          <rect
            :x="RIGHT_TRAVEL_X"
            :y="CHART_TOP - 12"
            :width="TRAVEL_W"
            :height="CHART_H + 24"
            class="cm-travel"
          />

          <!-- Curbs -->
          <line
            :x1="STREET_X"
            :x2="STREET_X"
            :y1="CHART_TOP - 10"
            :y2="CHART_BOTTOM + 10"
            class="cm-curb"
          />

          <line
            :x1="STREET_X + STREET_W"
            :x2="STREET_X + STREET_W"
            :y1="CHART_TOP - 10"
            :y2="CHART_BOTTOM + 10"
            class="cm-curb"
          />

          <!-- Parking / travel separators -->
          <line
            :x1="LEFT_TRAVEL_X"
            :x2="LEFT_TRAVEL_X"
            :y1="CHART_TOP - 10"
            :y2="CHART_BOTTOM + 10"
            class="cm-lane-line"
          />

          <line
            :x1="RIGHT_TRAVEL_X"
            :x2="RIGHT_TRAVEL_X"
            :y1="CHART_TOP - 10"
            :y2="CHART_BOTTOM + 10"
            class="cm-lane-line"
          />

          <line
            :x1="RIGHT_PARKING_X"
            :x2="RIGHT_PARKING_X"
            :y1="CHART_TOP - 10"
            :y2="CHART_BOTTOM + 10"
            class="cm-lane-line"
          />

          <!-- Centerline between travel lanes -->
          <line
            :x1="CENTER_X"
            :x2="CENTER_X"
            :y1="CHART_TOP - 10"
            :y2="CHART_BOTTOM + 10"
            class="cm-centerline"
          />
        </g>

        <!-- Zone labels -->
        <g>
          <text
            :x="LEFT_SIDEWALK_X + SIDEWALK_W / 2"
            :y="CHART_TOP - 22"
            text-anchor="middle"
            class="cm-zone-label"
          >
            SIDEWALK
          </text>

          <text
            :x="LEFT_PARKING_X + PARKING_W / 2"
            :y="CHART_TOP - 22"
            text-anchor="middle"
            class="cm-zone-label cm-zone-label-small"
          >
            PARKING
          </text>

          <text
            :x="CENTER_X"
            :y="CHART_TOP - 22"
            text-anchor="middle"
            class="cm-zone-label"
          >
            TRAVEL LANES
          </text>

          <text
            :x="RIGHT_PARKING_X + PARKING_W / 2"
            :y="CHART_TOP - 22"
            text-anchor="middle"
            class="cm-zone-label cm-zone-label-small"
          >
            PARKING
          </text>

          <text
            :x="RIGHT_SIDEWALK_X + SIDEWALK_W / 2"
            :y="CHART_TOP - 22"
            text-anchor="middle"
            class="cm-zone-label"
          >
            SIDEWALK
          </text>
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

.cm-street-base {
  fill: #f3f1ee;
  opacity: 0.95;
}

.cm-parking {
  fill: #ded8d1;
  opacity: 0.92;
}

.cm-travel {
  fill: #faf9f7;
  opacity: 0.98;
}

.cm-curb {
  stroke: #cfc8c0;
  stroke-width: 1.2;
}

.cm-lane-line {
  stroke: #ddd7d1;
  stroke-width: 1;
  stroke-dasharray: 4 7;
}

.cm-centerline {
  stroke: #bdb5ac;
  stroke-width: 1.2;
  stroke-dasharray: 6 7;
}

.cm-zone-label {
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 10px;
  letter-spacing: 0.08em;
  fill: var(--ghost, #aaa);
}

.cm-zone-label-small {
  font-size: 8px;
  letter-spacing: 0.06em;
}

.cm-dot {
  opacity: 0.94;
  stroke: var(--white, #fff);
  stroke-width: 1.4;
  transform-box: fill-box;
  transform-origin: center;
  animation: dotIn 0.32s ease both;
}

@keyframes dotIn {
  from {
    opacity: 0;
    transform: scale(0.72);
  }

  to {
    opacity: 0.94;
    transform: scale(1);
  }
}

.cm-dot-roadway {
  fill: var(--ink, #111111);
}

.cm-dot-sidewalk {
  fill: #6b665f;
}

.cm-dot-unknown {
  fill: var(--ghost, #aaa);
  opacity: 0.7;
}

.cm-state {
  padding: 24px;
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 12px;
  color: var(--ghost, #aaa);
}
</style>