<script setup>
const zones = [
  {
    label: 'Blocked movement',
    area: 'Sidewalk path',
    issues: ['Sidewalk blocked', 'Street blocked'],
  },
  {
    label: 'Physical setup',
    area: 'Curbside setup',
    issues: ['Setup condition', 'Barrier condition'],
  },
  {
    label: 'Permit and status',
    area: 'Program boundary',
    issues: ['Unauthorized', 'Restaurant status'],
  },
]
</script>

<template>
  <div class="street-conflict-diagram">
    <div class="street-conflict-kicker">
      Where the conflict appears
    </div>

    <div class="conflict-cards">
      <article
        v-for="zone in zones"
        :key="zone.label"
        class="conflict-card"
      >
        <div class="conflict-card-label">{{ zone.area }}</div>
        <h3>{{ zone.label }}</h3>
      </article>
    </div>
  </div>
</template>

<style scoped>
.street-conflict-diagram {
  width: 100%;
}

.street-conflict-kicker {
  margin-bottom: 18px;
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 12px;
  line-height: 1.3;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: var(--ghost, #a0a0a0);
}

.conflict-cards {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  border: 1px solid var(--rule, #e6e6e6);
  background: var(--white, #ffffff);
}

.conflict-card {
  min-height: 150px;
  padding: 28px 30px 30px;
  border-right: 1px solid var(--rule, #e6e6e6);
}

.conflict-card:last-child {
  border-right: none;
}

.conflict-card-label {
  margin-bottom: 14px;
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 11px;
  line-height: 1.3;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: var(--ghost, #a0a0a0);
}

.conflict-card h3 {
  margin: 0;
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: 24px;
  line-height: 1.2;
  letter-spacing: -0.035em;
  font-weight: 650;
  color: var(--ink, #111111);
}

@media (max-width: 900px) {
  .conflict-cards {
    grid-template-columns: 1fr;
  }

  .conflict-card {
    border-right: none;
    border-bottom: 1px solid var(--rule, #e6e6e6);
  }

  .conflict-card:last-child {
    border-bottom: none;
  }

  .conflict-card h3 {
    font-size: 22px;
  }
}
</style>