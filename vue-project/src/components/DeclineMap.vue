<script setup>
import { onMounted, onBeforeUnmount, ref } from 'vue'
import L from 'leaflet'
import 'leaflet/dist/leaflet.css'

const mapEl = ref(null)
let map = null

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
    const [historicRes, currentRes] = await Promise.all([
      fetch('/data/historic_manhattan_clean.geojson'),
      fetch('/data/thesis_points_final_v2.geojson'),
    ])

    if (!historicRes.ok || !currentRes.ok) {
      throw new Error('Failed to load GeoJSON files')
    }

    const historic = await historicRes.json()
    const current = await currentRes.json()

    // Historic = dim field
    const historicLayer = L.geoJSON(historic, {
      pointToLayer: (_, latlng) =>
        L.circleMarker(latlng, {
          radius: 2.8,
          fillColor: '#7f7f7f',
          color: '#7f7f7f',
          weight: 0,
          fillOpacity: 0.10,
          className: 'historic-dot',
          interactive: false,
        }),
    }).addTo(map)

    // Current = visible but not too dominant
    const currentLayer = L.geoJSON(current, {
      pointToLayer: (_, latlng) =>
        L.circleMarker(latlng, {
          radius: 1.8,
          fillColor: '#18140e',
          color: '#18140e',
          weight: 0,
          fillOpacity: 0.75,
          className: 'current-dot',
          interactive: false,
        }),
    }).addTo(map)

    // Fixed Manhattan framing
    const manhattanBounds = L.latLngBounds(
      [40.70, -74.02], // southwest
      [40.86, -73.93]  // northeast
    )

    map.fitBounds(manhattanBounds)
    
    // Log zoom level for reference
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
  background: #f6f6f6;
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