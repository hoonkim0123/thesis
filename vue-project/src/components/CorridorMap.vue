<script setup>
import { onMounted, onBeforeUnmount, ref } from 'vue'
import L from 'leaflet'
import 'leaflet/dist/leaflet.css'

const mapEl = ref(null)
const mapCaption = ref('Hover a street to isolate its corridor.')

let map = null
let bgLayer = null
let activeLineLayer = null
let activePointLayer = null
let pointsGeo = null
let streetsGeo = null
let pendingStreet = null

function highlightStreet(street) {
  if (!street) return

  if (!streetsGeo || !pointsGeo) {
    pendingStreet = street
    return
  }

  // clear previous
  if (activeLineLayer)  { map.removeLayer(activeLineLayer);  activeLineLayer = null }
  if (activePointLayer) { map.removeLayer(activePointLayer); activePointLayer = null }

  const label = street.corridor_label_clean

  // 1. corridor LINE segments
  const matchingLines = streetsGeo.features.filter(
    f => f.properties?.corridor_label_clean === label
  )

  if (matchingLines.length) {
    activeLineLayer = L.geoJSON(
      { type: 'FeatureCollection', features: matchingLines },
      {
        style: () => ({
          color: '#18140e',
          weight: 5,
          opacity: 0.95,
          lineCap: 'round',
          lineJoin: 'round',
        }),
      }
    ).addTo(map)
  }

  // 2. corridor POINTS on top of lines
  const matchingPoints = pointsGeo.features.filter(
    f => f.properties?.corridor_label_clean === label
  )

  if (matchingPoints.length) {
    activePointLayer = L.geoJSON(
      { type: 'FeatureCollection', features: matchingPoints },
      {
        pointToLayer: (_, latlng) =>
          L.circleMarker(latlng, {
            radius: 3.5,
            fillColor: '#18140e',
            fillOpacity: 0.95,
            color: 'transparent',
            weight: 0,
            interactive: false,
          }),
      }
    ).addTo(map)
  }

  // 3. pan to center
  const focusLayer = activeLineLayer || activePointLayer
  if (focusLayer) {
    const bounds = focusLayer.getBounds()
    if (bounds.isValid()) {
      map.panTo(bounds.getCenter(), { animate: true, duration: 0.3 })
    }
  }

  mapCaption.value = `${street.name} · ${street.n} locations`
}

onMounted(async () => {
  if (!mapEl.value) return

  map = L.map(mapEl.value, {
    center: [40.755, -73.985],
    zoom: 14,
    zoomControl: false,
    attributionControl: false,
    scrollWheelZoom: false,
    dragging: false,
    doubleClickZoom: false,
    boxZoom: false,
    keyboard: false,
    tap: false,
  })

  L.tileLayer(
    'https://{s}.basemaps.cartocdn.com/light_nolabels/{z}/{x}/{y}{r}.png',
    { subdomains: 'abcd', maxZoom: 19, opacity: 0.38, crossOrigin: true }
  ).addTo(map)

  try {
    const BASE = import.meta.env.BASE_URL
    const [pointsRes, streetsRes] = await Promise.all([
      fetch(`${BASE}data/corridor_points.geojson`),
      fetch(`${BASE}data/corridor_streets.geojson`),
])

    if (!pointsRes.ok || !streetsRes.ok) throw new Error('Failed to load corridor data')

    pointsGeo = await pointsRes.json()
    streetsGeo = await streetsRes.json()

    // background: all points very dim
    bgLayer = L.layerGroup()
    pointsGeo.features.forEach(f => {
      const [lon, lat] = f.geometry.coordinates
      L.circleMarker([lat, lon], {
        radius: 2,
        fillColor: '#18140e',
        fillOpacity: 0.07,
        color: 'transparent',
        weight: 0,
        interactive: false,
      }).addTo(bgLayer)
    })
    bgLayer.addTo(map)

    if (pendingStreet) {
      const nextStreet = pendingStreet
      pendingStreet = null
      highlightStreet(nextStreet)
    }

    setTimeout(() => map.invalidateSize(), 150)

  } catch (err) {
    console.error('CorridorMap error:', err)
  }
})

defineExpose({ highlightStreet })

onBeforeUnmount(() => {
  if (map) { map.remove(); map = null }
})
</script>

<template>
  <div class="corridor-map-container">
    <div ref="mapEl" class="s4-map"></div>
    <div class="map-footer">
      <div class="map-legend">
        <div class="leg-row">
          <div class="leg-line"></div>
          <span>Active corridor</span>
        </div>
        <div class="leg-row">
          <div class="leg-dot-light"></div>
          <span>Other locations</span>
        </div>
      </div>
      <div class="map-caption">{{ mapCaption }}</div>
    </div>
  </div>
</template>

<style scoped>
.corridor-map-container { width: 100%; }

.s4-map {
  width: 100%;
  height: 620px;
  border: 1px solid var(--rule);
  background: var(--off);
}

.map-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 10px;
}

.map-legend {
  display: flex;
  gap: 18px;
}

.leg-row {
  display: flex;
  align-items: center;
  gap: 7px;
  font-family: "IBM Plex Mono", monospace;
  font-size: 10px;
  color: var(--muted);
}

/* line icon — corridor */
.leg-line {
  width: 20px;
  height: 3px;
  background: var(--ink);
  border-radius: 2px;
  flex-shrink: 0;
}

/* dot icon — other */
.leg-dot-light {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: rgba(24, 20, 14, 0.2);
  flex-shrink: 0;
}

.map-caption {
  font-family: "IBM Plex Mono", monospace;
  font-size: 11px;
  color: var(--ghost);
  letter-spacing: 0.02em;
}

:deep(.leaflet-container) {
  background: var(--off);
  font-family: "IBM Plex Sans", sans-serif;
}

@media (max-width: 900px) {
  .s4-map { height: 440px; }
  .map-footer { flex-direction: column; align-items: flex-start; gap: 8px; }
}
</style>
