<script setup>
import { onMounted, ref } from 'vue'

const emit = defineEmits(['streetSelected'])

const STREETS = [
  { name: 'Amsterdam Avenue', area: 'Upper West Side', corridor_label_clean: 'Amsterdam Ave (Upper West Side)', n: 19 },
  { name: 'Columbus Avenue', area: 'Upper West Side', corridor_label_clean: 'Columbus Ave (Upper West Side)', n: 13 },
  { name: '2nd Avenue', area: 'Upper East Side', corridor_label_clean: '2nd Ave (Upper East Side)', n: 19 },
  { name: 'Mulberry Street', area: 'Little Italy / SoHo', corridor_label_clean: 'Mulberry St (Little Italy / SoHo)', n: 13 },
]

const activeStreet = ref('Amsterdam Avenue')

function selectStreet(name) {
  activeStreet.value = name
  emit('streetSelected', name)
}

onMounted(() => {
  selectStreet('Amsterdam Avenue')
})
</script>

<template>
  <div class="corridor-table-container">
    <div class="street-table">
      <button
        v-for="street in STREETS"
        :key="street.name"
        type="button"
        class="corridor-row"
        :class="{ active: activeStreet === street.name }"
        @click="selectStreet(street.name)"
      >
        <div class="st-left">
          <div class="st-name">{{ street.name }}</div>
          <div class="st-sub">{{ street.area }}</div>
        </div>
        <div class="st-right">
          <div class="st-count">{{ street.n }}</div>
        </div>
      </button>

      <div class="st-note">
        Other repeated streets include 3rd Ave, 1st Ave, and 8th Ave.
      </div>
    </div>
  </div>
</template>

<style scoped>
.corridor-table-container { width: 100%; }
.street-table { width: 100%; }

.corridor-row {
  width: 100%;
  appearance: none;
  border: 0;
  background: transparent;
  text-align: left;
  cursor: pointer;
  font: inherit;
}

.corridor-row {
  display: flex;
  align-items: flex-end;
  gap: 12px;
  padding: 14px 12px 14px 12px;
  border-bottom: 1px solid var(--rule);
  border-left: 2px solid transparent;
  transition: all 0.1s;
}

.corridor-row:hover {
  opacity: 0.7;
  background: var(--off);
}

.corridor-row.active {
  background: var(--accent-pale);
  border-left: 3px solid var(--accent);
  padding-left: 13px;
}

.corridor-row.active .st-name {
  color: var(--accent);
}

.st-left {
  flex: 1;
}

.st-name {
  font-size: 16px;
  font-weight: 600;
  color: var(--ink);
  line-height: 1.2;
}

.st-sub {
  font-size: 12px;
  color: var(--ghost);
  margin-top: 3px;
}

.st-right {
  display: flex;
  align-items: center;
  gap: 4px;
}

.st-count {
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
