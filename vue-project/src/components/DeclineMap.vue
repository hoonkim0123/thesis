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
let map = null
let historicLayer = null
let currentLayer = null

function setLayerStyle(layer, style) {
  if (!layer) return
  layer.eachLayer((dot) => {
    dot.setStyle(style)
  })
}

function updateHighlight(activeLayer) {
  if (!historicLayer || !currentLayer) return

  if (activeLayer === 'historic') {
    setLayerStyle(historicLayer, {
      radius: 2.6,
      fillColor: '#a6a6a6',
      color: '#a6a6a6',
      fillOpacity: 0.22,
      weight: 0,
    })

    setLayerStyle(currentLayer, {
      radius: 1.8,
      fillColor: '#111111',
      color: '#111111',
      fillOpacity: 0,
      weight: 0,
    })
  } else if (activeLayer === 'current') {
    setLayerStyle(historicLayer, {
      radius: 2.4,
      fillColor: '#a6a6a6',
      color: '#a6a6a6',
      fillOpacity: 0.06,
      weight: 0,
    })

    setLayerStyle(currentLayer, {
      radius: 2.2,
      fillColor: '#b44a3c',
      color: '#b44a3c',
      fillOpacity: 0.80,
      weight: 0,
    })
  } else {
    setLayerStyle(historicLayer, {
      radius: 2.6,
      fillColor: '#a6a6a6',
      color: '#a6a6a6',
      fillOpacity: 0.08,
      weight: 0,
    })

    setLayerStyle(currentLayer, {
      radius: 2.0,
      fillColor: '#111111',
      color: '#111111',
      fillOpacity: 0.70,
      weight: 0,
    })
  }
}

watch(
  () => props.activeLayer,
  (newValue) => {
    updateHighlight(newValue)
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
          radius: 2.6,
          fillColor: '#a6a6a6',
          color: '#a6a6a6',
          weight: 0,
          fillOpacity: 0.08,
          className: 'historic-dot',
          interactive: false,
        }),
    }).addTo(map)

    currentLayer = L.geoJSON(current, {
      pointToLayer: (_, latlng) =>
        L.circleMarker(latlng, {
          radius: 2.0,
          fillColor: '#111111',
          color: '#111111',
          weight: 0,
          fillOpacity: 0.70,
          className: 'current-dot',
          interactive: false,
        }),
    }).addTo(map)

    const manhattanBounds = L.latLngBounds(
      [40.70, -74.02],
      [40.86, -73.93]
    )

    map.fitBounds(manhattanBounds)

    updateHighlight(props.activeLayer)

    console.log('S2 DeclineMap zoom:', map.getZoom())
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
  <div ref="mapEl" class="decline-map-inner"></div>
</template>

<style scoped>
.decline-map-inner {
  width: 100%;
  height: 100%;
}

:deep(.leaflet-container) {
  font-family: 'IBM Plex Sans', sans-serif;
  background: var(--off);
}

:deep(.leaflet-tile) {
  opacity: 0.28;
}

:deep(.historic-dot) {
  filter: blur(0.8px);
}

:deep(.current-dot) {
  filter: none;
}
</style>