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
    lines: [
      'Outdoor dining still appears here as a continuous sequence along the street.',
      'Restaurant density and repeated frontage sustain that visibility.',
      'Elsewhere, this continuity breaks into isolated fragments.'
    ],
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
    lines: [
      'This corridor stretches across multiple neighborhoods.',
      'Sustained demand from both local and destination diners supports its length.',
      'Most streets no longer maintain this kind of continuity.'
    ],
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
    lines: [
      'Outdoor dining persists here as a secondary continuous corridor.',
      'Neighborhood demand and walkable access help maintain activity.',
      'Few areas sustain more than one such corridor.'
    ],
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
    lines: [
      'Outdoor dining here is tied to a strong street identity.',
      'As a destination street, it retains visibility despite overall decline.',
      'Outside places like this, that experience has largely disappeared.'
    ],
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
          <!-- Photo -->
          <div
            class="cg-image"
            :style="{
              backgroundImage: `url(${CORRIDORS[displayed()].image})`
            }"
          >
            <div class="cg-image-fallback">
              <span class="cg-fb-name">{{ CORRIDORS[displayed()].name }}</span>
              <span class="cg-fb-hint">Street photo coming soon</span>
            </div>
          </div>

          <!-- Street name overlay (top left) -->
          <div class="cg-overlay-title">
            {{ CORRIDORS[displayed()].name }}
          </div>

          <!-- Floating locator map (top right) -->
          <div class="cg-inset-overlay">
            <div ref="insetEl" class="cg-inset"></div>
          </div>

          <!-- Distribution strip label -->
          <div class="cg-strip-label">distribution along street</div>

          <!-- Distribution strip -->
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
            <div class="cg-caption-meta">
              <span class="cg-caption-type">{{ CORRIDORS[displayed()].character.toUpperCase() }}</span>
              <span class="cg-caption-count">{{ CORRIDORS[displayed()].count }} locations</span>
            </div>

            <div class="cg-caption-lines">
              <p v-for="(line, i) in CORRIDORS[displayed()].lines" :key="i" :class="{ 'is-contrast': i === 2 }">
                {{ line }}
              </p>
            </div>
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
  margin-top: var(--space-5);
  min-height: 480px;
}

/* ── Left list ── */
.cg-list {
  border-right: 1px solid var(--rule);
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
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
  display: none;
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
  background: rgba(248, 247, 244, 0.5);
  border-radius: 2px;
  z-index: 10;
  box-shadow: none;
  overflow: hidden;
  opacity: 0.7;
}

.cg-inset {
  width: 100%;
  height: 100%;
}

:deep(.cg-inset-overlay .leaflet-tile) {
  opacity: 0.4;
}

/* ── Street name overlay (top left) ── */
.cg-overlay-title {
  position: absolute;
  top: 28px;
  left: 28px;
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: 24px;
  font-weight: 700;
  color: rgba(255, 255, 255, 0.95);
  z-index: 11;
  letter-spacing: -0.01em;
}

/* ── Distribution strip ── */
.cg-strip-label {
  display: none;
}

.cg-distribution {
  position: absolute;
  bottom: 148px;
  left: 0;
  right: 0;
  height: 20px;
  display: flex;
  align-items: center;
  z-index: 9;
  padding: 0 24px;
  gap: 0;
}

.cg-dist-dot {
  position: absolute;
  width: 4px;
  height: 4px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.8);
  flex-shrink: 0;
}

/* Type-specific distribution styles */
.cg-distribution.is-continuous .cg-dist-dot {
  width: 3px;
  height: 3px;
  opacity: 0.9;
}

.cg-distribution.is-extended .cg-dist-dot {
  width: 2px;
  height: 2px;
  opacity: 0.6;
}

.cg-distribution.is-destination .cg-dist-dot {
  width: 4px;
  height: 4px;
  opacity: 1;
}

/* ── Caption (main info overlay) ── */
.cg-caption {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  padding: var(--space-4) var(--space-3) var(--space-3);
  background: linear-gradient(to top, rgba(24, 20, 14, 0.96) 0%, rgba(24, 20, 14, 0.88) 50%, rgba(24, 20, 14, 0.0) 100%);
  z-index: 5;
}

.cg-caption-meta {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
  margin-bottom: 10px;
}

.cg-caption-type {
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 0.04em;
  text-transform: uppercase;
  color: rgba(255, 255, 255, 0.72);
}

.cg-caption-count {
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: 13px;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.82);
}

.cg-caption-lines {
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: 16px;
  line-height: 1.5;
  font-weight: 500;
  color: rgba(255, 255, 255, 0.92);
}

.cg-caption-lines p {
  margin: 0 0 5px;
}

.cg-caption-lines p.is-contrast {
  font-size: 13px;
  color: rgba(255, 255, 255, 0.62);
  font-weight: 400;
  line-height: 1.45;
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

:deep(.cg-inset-overlay .leaflet-tile) {
  opacity: 0.3 !important;
}

/* ── Responsive ── */
@media (max-width: 900px) {
  .cg-wrap { grid-template-columns: 1fr; }
  .cg-list { border-right: none; border-bottom: 1px solid var(--rule); }
  .cg-image-wrap { min-height: 340px; }
  .cg-inset-overlay { width: 110px; height: 90px; }
  .cg-overlay-title { font-size: 20px; top: 20px; left: 20px; }
  .cg-distribution { bottom: 128px; }
}
</style>