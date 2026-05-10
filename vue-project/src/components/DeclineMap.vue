<script setup>
import { onMounted, onBeforeUnmount, ref, watch } from 'vue'
import L from 'leaflet'
import 'leaflet/dist/leaflet.css'

const props = defineProps({
  activeLayer: {
    type: String,
    default: null,
  },
})

const mapEl = ref(null)
const mapMode = ref('compare')

let map = null
let historicLayer = null
let currentLayer = null

function setLayerStyle(layer, style) {
  if (!layer) return

  layer.eachLayer((dot) => {
    if (dot.setStyle) {
      dot.setStyle(style)
    }
  })
}

function setCurrentMarkersVisible(visible) {
  if (!currentLayer) return

  currentLayer.eachLayer((marker) => {
    const el = marker.getElement()

    if (!el) return

    el.style.opacity = visible ? '1' : '0'
  })
}

function setMapMode(mode) {
  mapMode.value = mode
  updateHighlight(mode)
}

function updateHighlight(mode) {
  if (!historicLayer || !currentLayer) return

  if (mode === 'historic') {
    setLayerStyle(historicLayer, {
      radius: 1.8,
      fillColor: '#6f6a64',
      color: '#6f6a64',
      fillOpacity: 0.30,
      opacity: 0.30,
      weight: 0,
    })

    setCurrentMarkersVisible(false)
  } else if (mode === 'current') {
    setLayerStyle(historicLayer, {
      radius: 1.6,
      fillColor: '#b8b2ab',
      color: '#b8b2ab',
      fillOpacity: 0.025,
      opacity: 0.025,
      weight: 0,
    })

    setCurrentMarkersVisible(true)
  } else {
    setLayerStyle(historicLayer, {
      radius: 1.7,
      fillColor: '#8f8982',
      color: '#8f8982',
      fillOpacity: 0.11,
      opacity: 0.11,
      weight: 0,
    })

    setCurrentMarkersVisible(true)
  }
}

watch(
  () => props.activeLayer,
  (newValue) => {
    if (!newValue) return

    if (newValue === 'historic') {
      mapMode.value = 'historic'
      updateHighlight('historic')
    }

    if (newValue === 'current') {
      mapMode.value = 'current'
      updateHighlight('current')
    }
  }
)

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
    { maxZoom: 18 }
  ).addTo(map)

  try {
    const BASE = import.meta.env.BASE_URL

    const [historicRes, currentRes] = await Promise.all([
      fetch(`${BASE}data/historic_manhattan_clean.geojson`),
      fetch(`${BASE}data/thesis_points_final_v2.geojson`),
    ])

    if (!historicRes.ok || !currentRes.ok) {
      throw new Error('Failed to load GeoJSON files')
    }

    const historic = await historicRes.json()
    const current = await currentRes.json()

    historicLayer = L.geoJSON(historic, {
      pointToLayer: (_, latlng) =>
        L.circleMarker(latlng, {
          radius: 1.7,
          fillColor: '#8f8982',
          color: '#8f8982',
          weight: 0,
          fillOpacity: 0.11,
          opacity: 0.11,
          className: 'historic-dot',
          interactive: false,
        }),
    }).addTo(map)

    currentLayer = L.geoJSON(current, {
      pointToLayer: (_, latlng) =>
        L.marker(latlng, {
          icon: L.divIcon({
            className: 'current-shed-marker',
            html: '<span></span>',
            iconSize: [9, 9],
            iconAnchor: [4.5, 4.5],
          }),
          interactive: false,
        }),
    }).addTo(map)

    const manhattanBounds = L.latLngBounds(
      [40.70, -74.02],
      [40.86, -73.93]
    )

    map.fitBounds(manhattanBounds)

    updateHighlight(mapMode.value)
  } catch (error) {
    console.error('Error loading S2 decline map:', error)
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
  <div class="decline-map-shell" :class="`mode-${mapMode}`">
    <div class="map-toggle" aria-label="Map layer controls">
      <button
        type="button"
        :class="{ active: mapMode === 'historic' }"
        @click="setMapMode('historic')"
      >
        Peak
      </button>

      <button
        type="button"
        :class="{ active: mapMode === 'current' }"
        @click="setMapMode('current')"
      >
        Current
      </button>

      <button
        type="button"
        :class="{ active: mapMode === 'compare' }"
        @click="setMapMode('compare')"
      >
        Compare
      </button>
    </div>

    <div ref="mapEl" class="decline-map-inner"></div>
  </div>
</template>

<style scoped>
.decline-map-shell {
  position: relative;
  width: 100%;
  height: 100%;
  min-height: 520px;
  background: #f7f6f3;
  overflow: hidden;
}

.decline-map-inner {
  width: 100%;
  height: 100%;
  min-height: 520px;
}

.map-toggle {
  position: absolute;
  top: 18px;
  left: 18px;
  z-index: 500;
  display: flex;
  gap: 6px;
  padding: 5px;
  background: rgba(255, 255, 255, 0.88);
  border: 1px solid rgba(17, 17, 17, 0.08);
  border-radius: 999px;
  backdrop-filter: blur(8px);
  box-shadow: 0 8px 22px rgba(17, 17, 17, 0.06);
}

.map-toggle button {
  appearance: none;
  border: 0;
  border-radius: 999px;
  padding: 8px 13px;
  background: transparent;
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.09em;
  text-transform: uppercase;
  color: #8f8a84;
  cursor: pointer;
}

.map-toggle button.active {
  background: #111111;
  color: #ffffff;
}

.map-toggle button:focus {
  outline: none;
}

.map-toggle button:focus-visible {
  outline: 2px solid rgba(17, 17, 17, 0.22);
  outline-offset: 2px;
}

:deep(.leaflet-container) {
  font-family: "IBM Plex Sans", sans-serif;
  background: var(--off);
}

:deep(.leaflet-tile) {
  opacity: 0.18;
}

:deep(.historic-dot) {
  filter: none;
}

:deep(.current-shed-marker) {
  width: 9px;
  height: 9px;
  transition: opacity 0.22s ease;
}

:deep(.current-shed-marker span) {
  display: block;
  width: 6px;
  height: 6px;
  border-radius: 2px;
  background: #111111;
  opacity: 0.86;
  box-shadow:
    0 0 0 1px rgba(255, 255, 255, 0.72),
    0 1px 3px rgba(17, 17, 17, 0.20);
}

.mode-current :deep(.current-shed-marker span) {
  width: 7px;
  height: 7px;
  opacity: 0.92;
}

.mode-compare :deep(.current-shed-marker span) {
  width: 7px;
  height: 7px;
  border-radius: 2px;
  opacity: 0.88;
}

.mode-historic :deep(.current-shed-marker) {
  opacity: 0;
}
</style>