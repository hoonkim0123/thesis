<script setup>
import { onMounted, ref } from 'vue'

const rowsEl = ref(null)
const activeStreetName = ref(null)
const emit = defineEmits(['streetSelected'])

const STREETS = [
  { name: 'Amsterdam Avenue', area: 'Upper West Side', corridor_label_clean: 'Amsterdam Ave (Upper West Side)', n: 19 },
  { name: 'Columbus Avenue',  area: 'Upper West Side', corridor_label_clean: 'Columbus Ave (Upper West Side)', n: 13 },
  { name: '2nd Avenue',       area: 'Upper East Side', corridor_label_clean: '2nd Ave (Upper East Side)', n: 19 },
  { name: 'Mulberry Street',  area: 'Little Italy / SoHo', corridor_label_clean: 'Mulberry St (Little Italy / SoHo)', n: 13 },
]

function selectStreet(street) {
  activeStreetName.value = street.corridor_label_clean
  emit('streetSelected', street)

  if (!rowsEl.value) return
  rowsEl.value.querySelectorAll('.st-row').forEach(r => {
    r.classList.toggle('active', r.dataset.label === street.corridor_label_clean)
  })
}

function buildRows() {
  if (!rowsEl.value) return
  rowsEl.value.innerHTML = ''

  STREETS.forEach((s, i) => {
    const row = document.createElement('div')
    row.className = 'st-row' + (i === 0 ? ' active' : '')
    row.dataset.label = s.corridor_label_clean

    row.innerHTML = `
      <div class="st-left">
        <div class="st-name">${s.name}</div>
        <div class="st-sub">${s.area}</div>
      </div>
      <div class="st-right">
        <div class="st-count">${s.n}</div>
      </div>
    `

    row.addEventListener('mouseenter', () => selectStreet(s))
    rowsEl.value.appendChild(row)
  })
}

onMounted(() => {
  buildRows()
  // Amsterdam active by default
  selectStreet(STREETS[0])
})
</script>

<template>
  <div class="corridor-table-container">
    <div class="street-table">
      <div class="st-header">
        <div class="st-col">Street</div>
        <div class="st-col st-col-right">Locations</div>
      </div>
      <div ref="rowsEl"></div>
      <div class="st-note">
        Other repeated streets include 3rd Ave, 1st Ave, and 8th Ave.
      </div>
    </div>
  </div>
</template>

<style scoped>
.corridor-table-container { width: 100%; }
.street-table { width: 100%; }

.st-header {
  display: grid;
  grid-template-columns: 1fr 110px;
  padding: 0 0 10px;
  border-bottom: 1px solid var(--ink);
}

.st-col {
  font-family: "IBM Plex Mono", monospace;
  font-size: 10px;
  font-weight: 500;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: var(--ghost);
}

.st-col-right { text-align: right; }

:deep(.st-row) {
  display: grid;
  grid-template-columns: 1fr 110px;
  padding: 16px 8px 16px 0;
  border-bottom: 1px solid var(--rule);
  align-items: center;
  cursor: pointer;
  transition: background 0.1s;
}

:deep(.st-row:hover),
:deep(.st-row.active) {
  background: var(--off);
  padding-left: 8px;
}

:deep(.st-name) {
  font-size: 16px;
  font-weight: 600;
  color: var(--ink);
  line-height: 1.2;
}

:deep(.st-sub) {
  font-size: 12px;
  color: var(--ghost);
  margin-top: 3px;
}

:deep(.st-right) {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 7px;
}

:deep(.st-count) {
  font-family: "IBM Plex Mono", monospace;
  font-size: 24px;
  font-weight: 500;
  color: var(--ink);
  line-height: 1;
}

.st-note {
  font-family: "IBM Plex Mono", monospace;
  font-size: 11px;
  color: var(--ghost);
  padding-top: 14px;
  line-height: 1.5;
  letter-spacing: 0.02em;
}
</style>
