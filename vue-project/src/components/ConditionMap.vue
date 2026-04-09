<script setup>
import { onMounted, onBeforeUnmount, ref } from 'vue'
import L from 'leaflet'
import 'leaflet/dist/leaflet.css'

const mapEl = ref(null)
let map = null

function getPointStyle(feature) {
  const type = feature?.properties?.experience_type

  if (type === 'High access, low friction') {
    return {
      radius: 5.5,
      fillColor: '#4cd17d',
      color: '#ffffff',
      weight: 1,
      fillOpacity: 0.92,
      opacity: 1,
      className: 'point-high',
    }
  }

  if (type === 'Low access, high friction') {
    return {
      radius: 6,
      fillColor: '#ff6b6b',
      color: '#ffffff',
      weight: 1,
      fillOpacity: 0.95,
      opacity: 1,
      className: 'point-low',
    }
  }

  return {
    radius: 3.4,
    fillColor: '#cfcfcf',
    color: '#ffffff',
    weight: 0.7,
    fillOpacity: 0.32,
    opacity: 0.95,
    className: 'point-mixed',
  }
}

onMounted(async () => {
  if (!mapEl.value) return

  map = L.map(mapEl.value, {
    zoomControl: false,
    attributionControl: false,
    scrollWheelZoom: false,
    dragging: false,
    doubleClickZoom: false,
    boxZoom: false,
    keyboard: false,
    tap: false,
    preferCanvas: true,
  })

  L.tileLayer(
    'https://{s}.basemaps.cartocdn.com/light_nolabels/{z}/{x}/{y}{r}.png',
    {
      maxZoom: 18,
      opacity: 0.30,
    }
  ).addTo(map)

  try {
    const BASE = import.meta.env.BASE_URL
    const res = await fetch(`${BASE}data/thesis_points_final_v2.geojson`)
    if (!res.ok) throw new Error('Failed to load thesis_points_final_v2.geojson')

    const geojson = await res.json()

    const pointsLayer = L.geoJSON(geojson, {
      pointToLayer: (feature, latlng) =>
        L.circleMarker(latlng, {
          ...getPointStyle(feature),
          interactive: false,
        }),
    }).addTo(map)

    const manhattanBounds = L.latLngBounds(
      [40.70, -74.015],
      [40.86, -73.935]
    )

    map.fitBounds(manhattanBounds)
  } catch (err) {
    console.error('Error loading S6 condition map:', err)
  }
})

onBeforeUnmount(() => {
  if (map) {
    map.remove()
    map = null
  }
})
</script>

<template>
  <div class="condition-map-wrap">
    <div class="map-note">
      High access locations cluster in a few corridors. High friction locations appear more isolated.
    </div>
    <div ref="mapEl" class="condition-map"></div>
    <div class="condition-legend">
      <div class="legend-item">
        <span class="legend-dot legend-green"></span>
        <span>High access, low friction</span>
      </div>
      <div class="legend-item">
        <span class="legend-dot legend-red"></span>
        <span>Low access, high friction</span>
      </div>
      <div class="legend-item">
        <span class="legend-dot legend-gray"></span>
        <span>Mixed conditions</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
.condition-map-wrap {
  width: 100%;
}

.map-note {
  font-size: 13px;
  line-height: 1.5;
  color: #6f6f6f;
  margin-bottom: 12px;
  margin-top: 28px;
}

.condition-map {
  width: 100%;
  height: 640px;
  border: 1px solid #e6e6e6;
  background: #f6f6f6;
  margin-top: 28px;
}

.condition-legend {
  display: flex;
  flex-wrap: wrap;
  gap: 18px;
  margin-top: 8px;
  font-size: 12px;
  color: #666;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 8px;
}

.legend-dot {
  width: 11px;
  height: 11px;
  border-radius: 999px;
  display: inline-block;
}

.legend-green {
  background: #4cd17d;
}

.legend-red {
  background: #ff6b6b;
}

.legend-gray {
  background: #cfcfcf;
}

:deep(.leaflet-container) {
  font-family: 'IBM Plex Sans', sans-serif;
  background: #f6f6f6;
}

:deep(.leaflet-tile) {
  opacity: 0.30;
}
</style>
