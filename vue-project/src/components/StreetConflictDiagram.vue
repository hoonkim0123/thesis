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
  <div class="scd-wrap">
    <div class="scd-title">Where the conflict appears</div>

    <div class="scd-street">
      <div class="scd-zone scd-sidewalk">
        <span>Sidewalk</span>
      </div>

      <div class="scd-zone scd-parking">
        <span>Parking lane</span>
      </div>

      <div class="scd-zone scd-travel">
        <span>Travel lanes</span>
      </div>

      <div class="scd-zone scd-parking">
        <span>Parking lane</span>
      </div>

      <div class="scd-zone scd-sidewalk">
        <span>Sidewalk</span>
      </div>
    </div>

    <div class="scd-grid">
      <article
        v-for="zone in zones"
        :key="zone.label"
        class="scd-card"
      >
        <div class="scd-card-area">{{ zone.area }}</div>
        <div class="scd-card-title">{{ zone.label }}</div>

        <div class="scd-issues">
          <span
            v-for="issue in zone.issues"
            :key="issue"
            class="scd-pill"
          >
            {{ issue }}
          </span>
        </div>
      </article>
    </div>
  </div>
</template>

<style scoped>
.scd-wrap {
  width: 100%;
  margin: 40px 0 40px;
}

.scd-title {
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 11px;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--ghost, #a0a0a0);
  margin-bottom: 14px;
}

.scd-street {
  display: grid;
  grid-template-columns: 1fr 0.85fr 1.3fr 0.85fr 1fr;
  min-height: 56px;
  border: 1px solid var(--rule, #e6e6e6);
  background: var(--white, #ffffff);
}

.scd-zone {
  display: flex;
  align-items: center;
  justify-content: center;
  border-right: 1px solid var(--rule, #e6e6e6);
}

.scd-zone:last-child {
  border-right: none;
}

.scd-zone span {
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 9px;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: var(--ghost, #a0a0a0);
}

.scd-sidewalk {
  background: #f7f7f5;
}

.scd-parking {
  background: #ded8d1;
}

.scd-travel {
  background: #fbfaf8;
}

.scd-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  border-left: 1px solid var(--rule, #e6e6e6);
  border-right: 1px solid var(--rule, #e6e6e6);
  border-bottom: 1px solid var(--rule, #e6e6e6);
}

.scd-card {
  padding: 18px 20px 20px;
  border-right: 1px solid var(--rule, #e6e6e6);
  background: var(--white, #ffffff);
}

.scd-card:last-child {
  border-right: none;
}

.scd-card-area {
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 10px;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: var(--ghost, #a0a0a0);
  margin-bottom: 8px;
}

.scd-card-title {
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: 18px;
  line-height: 1.2;
  letter-spacing: -0.025em;
  font-weight: 650;
  color: var(--ink, #111111);
  margin-bottom: 14px;
}

.scd-issues {
  display: flex;
  flex-wrap: wrap;
  gap: 7px;
}

.scd-pill {
  display: inline-flex;
  align-items: center;
  padding: 5px 8px;
  background: var(--accent-pale, #faefed);
  color: var(--accent-dark, #7c2d24);
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 10px;
  letter-spacing: 0.02em;
}

@media (max-width: 760px) {
  .scd-street {
    grid-template-columns: 1fr;
  }

  .scd-zone {
    min-height: 38px;
    border-right: none;
    border-bottom: 1px solid var(--rule, #e6e6e6);
  }

  .scd-zone:last-child {
    border-bottom: none;
  }

  .scd-grid {
    grid-template-columns: 1fr;
  }

  .scd-card {
    border-right: none;
    border-bottom: 1px solid var(--rule, #e6e6e6);
  }

  .scd-card:last-child {
    border-bottom: none;
  }
}
</style>