<script setup>
import { onMounted, ref, watch } from 'vue'

const rowsEl = ref(null)
const activeStreetName = ref(null)
const emit = defineEmits(['streetSelected'])

const props = defineProps({
  hoveredCorridor: {
    type: String,
    default: null,
  },
})

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
    row.addEventListener('mouseleave', () => {
      if (!props.hoveredCorridor && activeStreetName.value === s.corridor_label_clean) {
        selectStreet(STREETS[0])
      }
    })
    rowsEl.value.appendChild(row)
  })
}

// Watch for point hover from map
watch(
  () => props.hoveredCorridor,
  (newValue) => {
    if (!newValue) return
    
    const street = STREETS.find(s => s.corridor_label_clean === newValue)
    if (street) {
      if (rowsEl.value) {
        rowsEl.value.querySelectorAll('.st-row').forEach(r => {
          r.classList.toggle('active', r.dataset.label === newValue)
        })
      }
      activeStreetName.value = newValue
    }
  }
)

onMounted(() => {
  buildRows()
  // Amsterdam active by default
  selectStreet(STREETS[0])
})
</script>

<template>
  <div class="corridor-table-container">
    <div class="street-table">
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

:deep(.st-row) {
  display: flex;
  align-items: flex-end;
  gap: 12px;
  padding: 14px 12px 14px 12px;
  border-bottom: 1px solid var(--rule);
  border-left: 2px solid transparent;
  cursor: pointer;
  transition: all 0.1s;
}

:deep(.st-row:hover) {
  opacity: 0.7;
}

:deep(.st-row.active) {
  background: transparent;
  border-left-color: transparent;
}

:deep(.st-left) {
  flex: 1;
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
  align-items: center;
  gap: 4px;
}

:deep(.st-count) {
  font-family: "IBM Plex Mono", monospace;
  font-size: 11px;
  color: var(--ghost);
  line-height: 1;
}

.st-note {
  font-family: "IBM Plex Mono", monospace;
  font-size: 11px;
  color: var(--ghost);
  padding-top: 10px;
  line-height: 1.5;
  letter-spacing: 0.02em;
}
</style>
