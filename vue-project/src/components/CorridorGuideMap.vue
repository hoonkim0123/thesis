<script setup>
import { onMounted, onBeforeUnmount, ref, watch } from 'vue'
import L from 'leaflet'
import 'leaflet/dist/leaflet.css'

const insetEl = ref(null)
const activeIndex = ref(null)   // click-locked
const hoverIndex = ref(null)    // hover preview
const corridorDistributions = ref({})   // position data for strips

let insetMap = null
let insetMarkers = []
let highlightLayer = null

const CORRIDORS = [
  {
    name: 'Amsterdam Avenue',
    area: 'Upper West Side',
    count: 18,
    character: 'Continuous corridor',
    interpretation: 'One of few streets where outdoor dining still appears as a continuous pattern.',
    contrast: 'Elsewhere, outdoor dining exists only as isolated remnants.',
    image: '/images/corridor-amsterdam.jpg',
    center: [40.7831, -73.9812],
    matchKey: 'AMSTERDAM',
    zoom: 15,
  },
  {
    name: '2nd Avenue',
    area: 'East Village · Midtown East',
    count: 15,
    character: 'Extended corridor',
    interpretation: 'A long corridor sustaining demand from both local and destination diners.',
    contrast: 'Most streets no longer span neighborhoods this way.',
    image: '/images/corridor-2ave.jpg',
    center: [40.7282, -73.9855],
    matchKey: '2 AVENUE',
    zoom: 15,
  },
  {
    name: 'Columbus Avenue',
    area: 'Upper West Side',
    count: 13,
    character: 'Continuous corridor',
    interpretation: 'The Upper West Side retains two active corridors—a density few neighborhoods match.',
    contrast: 'Most areas do not reach this level of concentration.',
    image: '/images/corridor-columbus.jpg',
    center: [40.7794, -73.9800],
    matchKey: 'COLUMBUS',
    zoom: 15,
  },
  {
    name: 'Mulberry Street',
    area: 'Little Italy',
    count: 13,
    character: 'Destination corridor',
    interpretation: 'A destination street where outdoor dining still reads as part of street identity.',
    contrast: 'This street-level experience has largely disappeared elsewhere.',
    image: '/images/corridor-mulberry.jpg',
    center: [40.7195, -73.9973],
    matchKey: 'MULBERRY',
    zoom: 15,
  },
]

// Which corridor is currently displayed (hover overrides click for preview)
const displayed = () => hoverIndex.value !== null ? hoverIndex.value : activeIndex.value

function selectCorridor(i) {
  activeIndex.value = activeIndex.value === i ? null : i
  updateInset()
}

function onHover(i) {
  hoverIndex.value = i
  updateInset()
}

function onUnhover() {
  hoverIndex.value = null
  updateInset()
}

// ── Calculate actual distribution positions ──
function calculateDistributions() {
  if (!allPoints) return
  
  const distributions = {}
  CORRIDORS.forEach((c, idx) => {
    const matching = allPoints.features.filter(f =>
      (f.properties?.street_name || '').toUpperCase().includes(c.matchKey)
    )
    
    if (matching.length === 0) {
      distributions[idx] = []
      return
    }
    
    // Extract longitudes and normalize to 0-100%
    const lngs = matching.map(f => f.geometry.coordinates[0]).sort((a, b) => a - b)
    const minLng = Math.min(...lngs)
    const maxLng = Math.max(...lngs)
    const range = maxLng - minLng || 1
    
    distributions[idx] = lngs.map(lng => ((lng - minLng) / range * 100))
  })
  
  corridorDistributions.value = distributions
}

// ── Get corridor type for CSS ──
function getCorridorType(character) {
  if (character.includes('Continuous')) return 'continuous'
  if (character.includes('Extended')) return 'extended'
  if (character.includes('Destination')) return 'destination'
  return 'default'
}

// ── inset map ──────────────────────────────────────────────
let allPoints = null   // cache geo

async function initInset() {
  if (!insetEl.value) return

  insetMap = L.map(insetEl.value, {
    center: [40.748, -73.984],
    zoom: 11,
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
    { subdomains: 'abcd', maxZoom: 19, opacity: 0.3, crossOrigin: true }
  ).addTo(insetMap)

  // Load background points
  try {
    const BASE = import.meta.env.BASE_URL
    const res = await fetch(`${BASE}data/thesis_points_final_v2.geojson`)
    if (res.ok) {
      allPoints = await res.json()
      calculateDistributions()  // ← calculate after load
      L.geoJSON(allPoints, {
        pointToLayer: (_, latlng) =>
          L.circleMarker(latlng, {
            radius: 2.5,
            fillColor: '#18140e',
            fillOpacity: 0.08,
            color: 'transparent',
            weight: 0,
            interactive: false,
          }),
      }).addTo(insetMap)
    }
  } catch (e) { /* silent */ }

  // Corridor dots (always visible, dim)
  CORRIDORS.forEach((c, i) => {
    const m = L.circleMarker(c.center, {
      radius: 6,
      fillColor: '#18140e',
      fillOpacity: 0.25,
      color: '#ffffff',
      weight: 1.5,
      interactive: false,
    }).addTo(insetMap)
    insetMarkers.push(m)
  })

  setTimeout(() => insetMap.invalidateSize(), 100)
}

function updateInset() {
  if (!insetMap) return
  const d = displayed()

  // Reset all markers
  insetMarkers.forEach((m, i) => {
    m.setStyle({
      fillOpacity: d === null || d === i ? (d === i ? 1 : 0.15) : 0.1,
      radius: d === i ? 8 : 5,
      fillColor: d === i ? '#18140e' : '#18140e',
    })
  })

  // Highlight layer
  if (highlightLayer) { insetMap.removeLayer(highlightLayer); highlightLayer = null }

  if (d !== null && allPoints) {
    const c = CORRIDORS[d]
    const matching = allPoints.features.filter(f =>
      (f.properties?.street_name || '').toUpperCase().includes(c.matchKey)
    )
    highlightLayer = L.geoJSON(
      { type: 'FeatureCollection', features: matching },
      {
        pointToLayer: (_, latlng) =>
          L.circleMarker(latlng, {
            radius: 5,
            fillColor: '#18140e',
            fillOpacity: 1,
            color: '#ffffff',
            weight: 1.5,
            interactive: false,
          }),
      }
    ).addTo(insetMap)

    // Pan inset to corridor center
    insetMap.setView(CORRIDORS[d].center, CORRIDORS[d].zoom, { animate: true, duration: 0.4 })
  } else {
    insetMap.setView([40.748, -73.984], 11, { animate: true, duration: 0.4 })
  }
}

onMounted(() => initInset())
onBeforeUnmount(() => { if (insetMap) { insetMap.remove(); insetMap = null } })
</script>

<template>
  <div class="cg-wrap">

    <!-- LEFT: corridor list -->
    <div class="cg-list">
      <button
        v-for="(c, i) in CORRIDORS"
        :key="i"
        class="cg-item"
        :class="{
          'is-active': activeIndex === i,
          'is-hover':  hoverIndex === i && activeIndex === null,
        }"
        @click="selectCorridor(i)"
        @mouseenter="onHover(i)"
        @mouseleave="onUnhover()"
      >
        <div class="cg-item-top">
          <span class="cg-name">{{ c.name }}</span>
          <span class="cg-count">{{ c.count }}</span>
        </div>
        <div class="cg-area">{{ c.area }}</div>
        <div class="cg-char">{{ c.character }}</div>
      </button>

      <div class="cg-hint">
        {{ activeIndex !== null ? 'Click again to deselect' : 'Select a corridor' }}
      </div>
    </div>

    <!-- RIGHT: image panel with overlays -->
    <div class="cg-right">

      <!-- Image area (with floating map + info layers) -->
      <div class="cg-image-wrap">
        <template v-if="displayed() !== null">
          <!-- Photo (swap with real image when available) -->
          <div
            class="cg-image"
            :style="{
              backgroundImage: `url(${CORRIDORS[displayed()].image})`
            }"
          >
            <!-- Fallback shown when image not yet loaded or missing -->
            <div class="cg-image-fallback">
              <span class="cg-fb-name">{{ CORRIDORS[displayed()].name }}</span>
              <span class="cg-fb-hint">Street photo coming soon</span>
            </div>
          </div>

          <!-- Floating locator map (top right) -->
          <div class="cg-inset-overlay">
            <div ref="insetEl" class="cg-inset"></div>
          </div>

          <!-- Distribution strip (under map) -->
          <div 
            class="cg-distribution"
            :class="`is-${getCorridorType(CORRIDORS[displayed()].character)}`"
          >
            <div 
              v-for="(pos, i) in corridorDistributions[displayed()]" 
              :key="i"
              class="cg-dist-dot"
              :style="{ left: pos + '%' }"
            ></div>
          </div>

          <!-- Main info overlay at bottom -->
          <div class="cg-caption">
            <div class="cg-caption-street">
              <span class="cg-street-name">{{ CORRIDORS[displayed()].name }}</span>
              <span class="cg-street-count">{{ CORRIDORS[displayed()].count }}</span>
            </div>
            <div class="cg-caption-type">{{ CORRIDORS[displayed()].character }}</div>
            <p class="cg-cap-text">{{ CORRIDORS[displayed()].interpretation }}</p>
            <p class="cg-cap-contrast">{{ CORRIDORS[displayed()].contrast }}</p>
          </div>
        </template>

        <!-- Placeholder when nothing selected/hovered -->
        <template v-else>
          <div class="cg-placeholder">
            <span>Hover or select a corridor to see its street-level character.</span>
          </div>
        </template>
      </div>

    </div>
  </div>
</template>

<style scoped>
.cg-wrap {
  display: grid;
  grid-template-columns: 260px 1fr;
  border: 1px solid var(--rule);
  margin-top: 40px;
  min-height: 480px;
}

/* ── Left list ── */
.cg-list {
  border-right: 1px solid var(--rule);
  display: flex;
  flex-direction: column;
}

.cg-item {
  display: block;
  width: 100%;
  text-align: left;
  padding: 16px 20px;
  border: none;
  border-bottom: 1px solid var(--rule);
  background: transparent;
  cursor: pointer;
  transition: background 0.15s;
  flex-shrink: 0;
}

.cg-item:hover,
.cg-item.is-hover  { background: var(--off, #f8f7f4); }
.cg-item.is-active { background: var(--ink, #18140e); }

.cg-item.is-active .cg-name,
.cg-item.is-active .cg-area,
.cg-item.is-active .cg-count,
.cg-item.is-active .cg-char { color: #fff; }

.cg-item-top {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
  margin-bottom: 3px;
}

.cg-name {
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: 13px;
  font-weight: 600;
  color: var(--ink, #18140e);
}

.cg-count {
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 12px;
  font-weight: 700;
  color: var(--ink, #18140e);
}

.cg-area {
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 10px;
  color: var(--muted, #888);
  margin-bottom: 4px;
}

.cg-char {
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 9px;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  color: var(--ghost, #bbb);
}

.cg-hint {
  margin-top: auto;
  padding: 14px 20px;
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 9px;
  color: var(--ghost, #ccc);
  border-top: 1px solid var(--rule);
}

/* ── Right panel ── */
.cg-right {
  display: flex;
  flex-direction: column;
  flex: 1;
}

/* Image area (unified with all overlays) */
.cg-image-wrap {
  position: relative;
  flex: 1;
  min-height: 480px;
  background: var(--off, #f8f7f4);
  display: flex;
  align-items: stretch;
}

.cg-image {
  position: absolute;
  inset: 0;
  background-size: cover;
  background-position: center;
  background-color: #e8e6e0;
}

.cg-image-fallback {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 8px;
  background: #edecea;
  z-index: 0;
}

.cg-image[style*="url("] .cg-image-fallback {
  display: none;
}

.cg-fb-name {
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: 15px;
  font-weight: 600;
  color: var(--ink, #18140e);
}

.cg-fb-hint {
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 10px;
  color: var(--ghost, #bbb);
}

/* ── Floating inset map (overlay) ── */
.cg-inset-overlay {
  position: absolute;
  top: 16px;
  right: 16px;
  width: 140px;
  height: 120px;
  border: none;
  background: rgba(248, 247, 244, 0.65);
  border-radius: 3px;
  z-index: 10;
  box-shadow: none;
  overflow: hidden;
}

.cg-inset {
  width: 100%;
  height: 100%;
}

/* ── Distribution strip ── */
.cg-distribution {
  position: absolute;
  bottom: 148px;
  left: 0;
  right: 0;
  height: 24px;
  display: flex;
  align-items: center;
  z-index: 9;
  padding: 0 28px;
  gap: 2px;
}

.cg-dist-dot {
  position: absolute;
  width: 4px;
  height: 4px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.7);
  flex-shrink: 0;
  transition: opacity 0.2s ease;
}

/* Type-specific styles */
.cg-distribution.is-continuous .cg-dist-dot {
  width: 5px;
  height: 5px;
  opacity: 0.8;
}

.cg-distribution.is-extended .cg-dist-dot {
  width: 3px;
  height: 3px;
  opacity: 0.6;
}

.cg-distribution.is-destination .cg-dist-dot {
  width: 6px;
  height: 6px;
  opacity: 0.9;
}

/* ── Caption (main info overlay) ── */
.cg-caption {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 28px 28px 24px;
  background: linear-gradient(to top, rgba(24, 20, 14, 0.95) 0%, rgba(24, 20, 14, 0.85) 50%, rgba(24, 20, 14, 0.0) 100%);
  z-index: 5;
}

.cg-caption-street {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
  margin-bottom: 8px;
}

.cg-street-name {
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: 20px;
  font-weight: 700;
  color: #ffffff;
  letter-spacing: -0.01em;
}

.cg-street-count {
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 12px;
  font-weight: 700;
  color: rgba(255, 255, 255, 0.8);
}

.cg-caption-type {
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 9px;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: rgba(255, 255, 255, 0.6);
  margin-bottom: 8px;
}

.cg-cap-text {
  font-family: var(--serif, "EB Garamond", serif);
  font-size: 14px;
  line-height: 1.5;
  color: rgba(255, 255, 255, 0.88);
  margin: 0 0 5px;
}

.cg-cap-contrast {
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 9px;
  color: rgba(255, 255, 255, 0.5);
  line-height: 1.4;
  margin: 0;
}

/* ── Placeholder ── */
.cg-placeholder {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 11px;
  color: var(--ghost, #bbb);
  text-align: center;
  padding: 32px;
}

/* ── Leaflet ── */
:deep(.leaflet-container) {
  background: var(--off, #f8f7f4);
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
}

/* ── Responsive ── */
@media (max-width: 900px) {
  .cg-wrap { grid-template-columns: 1fr; }
  .cg-list { border-right: none; border-bottom: 1px solid var(--rule); }
  .cg-image-wrap { min-height: 340px; }
  .cg-inset-overlay { width: 120px; height: 100px; }
  .cg-street-name { font-size: 18px; }
  .cg-distribution { bottom: 128px; }
}
</style>