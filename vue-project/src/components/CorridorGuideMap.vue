<script setup>
import { onMounted, onBeforeUnmount, ref } from 'vue'
import L from 'leaflet'
import 'leaflet/dist/leaflet.css'

const mapEl = ref(null)
let map = null

// 4 corridors — coordinates + count from EDA
const CORRIDORS = [
  {
    name: 'Amsterdam Avenue',
    area: 'Upper West Side',
    count: 18,
    center: [40.7831, -73.9812],
    color: '#18140e',
  },
  {
    name: '2nd Avenue',
    area: 'East Village · Midtown East',
    count: 15,
    center: [40.7282, -73.9855],
    color: '#18140e',
  },
  {
    name: 'Columbus Avenue',
    area: 'Upper West Side',
    count: 13,
    center: [40.7794, -73.9800],
    color: '#18140e',
  },
  {
    name: 'Mulberry Street',
    area: 'Little Italy',
    count: 13,
    center: [40.7195, -73.9973],
    color: '#18140e',
  },
]

onMounted(async () => {
  if (!mapEl.value) return

  map = L.map(mapEl.value, {
    center: [40.748, -73.984],
    zoom: 12,
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

  // Load thesis_points_final_v2 — all 318 current locations as dim background
  try {
    const BASE = import.meta.env.BASE_URL
    const res = await fetch(`${BASE}data/thesis_points_final_v2.geojson`)
    if (res.ok) {
      const geo = await res.json()
      L.geoJSON(geo, {
        pointToLayer: (_, latlng) =>
          L.circleMarker(latlng, {
            radius: 3,
            fillColor: '#18140e',
            fillOpacity: 0.08,
            color: 'transparent',
            weight: 0,
            interactive: false,
          }),
      }).addTo(map)
    }
  } catch (e) {
    console.warn('Could not load thesis_points_final_v2:', e)
  }

  // Draw corridor markers + labels
  CORRIDORS.forEach((c) => {
    // Large circle marker
    L.circleMarker(c.center, {
      radius: 10,
      fillColor: c.color,
      fillOpacity: 1,
      color: '#ffffff',
      weight: 2,
      interactive: false,
    }).addTo(map)

    // Count label inside circle (DivIcon)
    L.marker(c.center, {
      icon: L.divIcon({
        className: '',
        html: `<div class="guide-count">${c.count}</div>`,
        iconSize: [20, 20],
        iconAnchor: [10, 10],
      }),
      interactive: false,
    }).addTo(map)

    // Street name label offset to the right
    L.marker(c.center, {
      icon: L.divIcon({
        className: '',
        html: `
          <div class="guide-label">
            <div class="guide-name">${c.name}</div>
            <div class="guide-area">${c.area}</div>
          </div>`,
        iconSize: [160, 40],
        iconAnchor: [-16, 20],
      }),
      interactive: false,
    }).addTo(map)
  })

  setTimeout(() => map.invalidateSize(), 150)
})

onBeforeUnmount(() => {
  if (map) { map.remove(); map = null }
})
</script>

<template>
  <div class="guide-map-wrap">
    <div ref="mapEl" class="guide-map"></div>
    <div class="guide-footer">
      <div class="guide-legend-row">
        <div class="guide-dot"></div>
        <span>Active corridor</span>
      </div>
      <div class="guide-legend-row">
        <div class="guide-dot-dim"></div>
        <span>All 318 current locations</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
.guide-map-wrap {
  width: 100%;
  margin-top: 40px;
}

.guide-map {
  width: 100%;
  height: 520px;
  border: 1px solid var(--rule);
  background: var(--off);
}

.guide-footer {
  display: flex;
  gap: 24px;
  margin-top: 10px;
  flex-wrap: wrap;
}

.guide-legend-row {
  display: flex;
  align-items: center;
  gap: 7px;
  font-family: "IBM Plex Mono", monospace;
  font-size: 10px;
  color: var(--muted);
}

.guide-dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: #18140e;
  flex-shrink: 0;
}

.guide-dot-dim {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: rgba(24, 20, 14, 0.2);
  flex-shrink: 0;
}

:deep(.leaflet-container) {
  background: var(--off);
  font-family: "IBM Plex Sans", sans-serif;
}

:deep(.guide-count) {
  width: 20px;
  height: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: "IBM Plex Mono", monospace;
  font-size: 9px;
  font-weight: 700;
  color: #ffffff;
  pointer-events: none;
}

:deep(.guide-label) {
  padding-left: 6px;
  pointer-events: none;
}

:deep(.guide-name) {
  font-family: "IBM Plex Sans", sans-serif;
  font-size: 11px;
  font-weight: 600;
  color: #18140e;
  white-space: nowrap;
  line-height: 1.4;
}

:deep(.guide-area) {
  font-family: "IBM Plex Mono", monospace;
  font-size: 9px;
  color: #888;
  white-space: nowrap;
  line-height: 1.4;
}

@media (max-width: 900px) {
  .guide-map { height: 400px; }
}
</style>
