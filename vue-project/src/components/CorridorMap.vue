<script setup>
import { onMounted, onBeforeUnmount, ref } from 'vue'
import L from 'leaflet'
import 'leaflet/dist/leaflet.css'

const mapEl = ref(null)
const mapCaption = ref('Hover a street to see where it appears.')
const tooltipText = ref('')
const tooltipX = ref(0)
const tooltipY = ref(0)
const showTooltip = ref(false)

const emit = defineEmits(['pointHover', 'pointLeave'])

let map = null
let bgLayer = null
let activeLineLayer = null
let activePointLayer = null
let pointsGeo = null
let streetsGeo = null
let pendingStreet = null

function getTypeLabel(licenseType) {
  const typeMap = {
    'Roadway': 'roadway dining',
    'Sidewalk': 'sidewalk dining',
  }
  return typeMap[licenseType] || licenseType
}

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
          color: '#bbbbbb',
          weight: 1,
          opacity: 0.5,
          lineCap: 'round',
          lineJoin: 'round',
        }),
      }
    ).addTo(map)
  }

  // 2. corridor POINTS on top of lines with hover interaction
  const matchingPoints = pointsGeo.features.filter(
    f => f.properties?.corridor_label_clean === label
  )

  if (matchingPoints.length) {
    activePointLayer = L.geoJSON(
      { type: 'FeatureCollection', features: matchingPoints },
      {
        pointToLayer: (feature, latlng) => {
          const marker = L.circleMarker(latlng, {
            radius: 5,
            fillColor: '#111111',
            fillOpacity: 0.8,
            color: '#ffffff',
            weight: 1,
            interactive: true,
          })

          marker.on('mouseover', (e) => {
            const props = feature.properties
            const corridor = props.corridor_label_clean
            const type = getTypeLabel(props.license_type)
            
            // Extract neighborhood from corridor_label_clean (format: "Street (Neighborhood)")
            const match = corridor.match(/(.+?)\s*\((.+?)\)/)
            const neighborhood = match ? match[2] : ''
            const streetName = match ? match[1] : corridor

            tooltipText.value = `${streetName}<br>${neighborhood} · ${type}`
            tooltipX.value = e.containerPoint.x
            tooltipY.value = e.containerPoint.y
            showTooltip.value = true

            emit('pointHover', corridor)
          })

          marker.on('mouseout', () => {
            showTooltip.value = false
            emit('pointLeave')
          })

          return marker
        },
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
      fetch(`${BASE}data/corridor_points_labeled.geojson`),
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
    
    <!-- Tooltip -->
    <div v-if="showTooltip" class="map-tooltip" :style="{ left: tooltipX + 'px', top: tooltipY + 'px' }">
      <div v-html="tooltipText"></div>
    </div>

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
.corridor-map-container {
  width: 100%;
  position: relative;
}

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
  padding-right: 12px;
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
  height: 2px;
  background: #bbbbbb;
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

/* Tooltip */
.map-tooltip {
  position: absolute;
  background: rgba(17, 17, 17, 0.95);
  color: #ffffff;
  padding: 6px 10px;
  border-radius: 2px;
  font-family: "IBM Plex Sans", sans-serif;
  font-size: 11px;
  line-height: 1.4;
  white-space: nowrap;
  pointer-events: none;
  z-index: 1000;
  transform: translateX(-50%) translateY(-100%);
  margin-top: -8px;
  letter-spacing: 0.01em;
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
