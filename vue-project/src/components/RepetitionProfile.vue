<template>
  <div class="model-signal">
    <div class="signal-frame">
      <div class="signal-panel map-panel">
        <div class="panel-kicker">Map pattern</div>

        <h3>Remaining locations repeat on fewer streets.</h3>

        <div class="mini-map" aria-hidden="true">
          <div class="street-line street-line-main"></div>
          <div class="street-line street-line-side street-line-a"></div>
          <div class="street-line street-line-side street-line-b"></div>

          <span class="map-point point-a"></span>
          <span class="map-point point-b"></span>
          <span class="map-point point-c"></span>
          <span class="map-point point-d"></span>
          <span class="map-point point-e"></span>
          <span class="map-point point-f"></span>
        </div>

        <p>
          They are not evenly spread across Manhattan. They appear again and again along a smaller set of streets.
        </p>
      </div>

      <div class="signal-connector" aria-hidden="true">
        <span></span>
      </div>

      <div class="signal-panel model-panel">
        <div class="panel-kicker">Model signal</div>

        <h3>Same street repetition</h3>

        <div class="signal-badge">
          Strongest signal tested
        </div>

        <div class="model-signal-stack" aria-hidden="true">
          <div class="signal-rank signal-rank-main">
            <span>01</span>
            <strong>Same street repetition</strong>
          </div>
          <div class="signal-rank">
            <span>02</span>
            <strong>Nearby outdoor dining</strong>
          </div>
          <div class="signal-rank">
            <span>03</span>
            <strong>Street context</strong>
          </div>
        </div>

        <p>
          The model read this as the strongest signal among the features tested.
        </p>
      </div>
    </div>

    <div class="supporting-signals">
      <div class="supporting-head">
        <span>Supporting signals tested</span>
      </div>

      <div class="signal-list">
        <span>Nearby outdoor dining</span>
        <span>Subway distance</span>
        <span>Street width</span>
        <span>Corridor presence</span>
        <span>Transit proximity</span>
      </div>
    </div>

    <p class="signal-note">
      Random Forest feature importance. Exploratory, not causal.
    </p>
  </div>
</template>

<script setup>
</script>

<style scoped>
.model-signal {
  margin: 38px 0 0;
  padding: 28px 0 0;
  border-top: 1px solid rgba(17, 17, 17, 0.10);
  border-bottom: none;
}

.signal-frame {
  display: grid;
  grid-template-columns: 1fr 72px 1fr;
  align-items: stretch;
}

.signal-panel {
  min-height: 320px;
  padding: 28px 30px 30px;
  background: #fafaf8;
  border: 1px solid rgba(17, 17, 17, 0.10);
  display: flex;
  flex-direction: column;
}

.model-panel {
  background: #2f2b27;
  border-color: #2f2b27;
  color: #ffffff;
}

.panel-kicker {
  margin-bottom: 24px;
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: #9b948d;
}

.model-panel .panel-kicker {
  color: rgba(255, 255, 255, 0.52);
}

.signal-panel h3 {
  max-width: 440px;
  margin: 0;
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: clamp(32px, 3.4vw, 52px);
  line-height: 1.02;
  letter-spacing: -0.055em;
  font-weight: 720;
  color: #111111;
}

.model-panel h3 {
  color: #ffffff;
}

.signal-panel p {
  margin: auto 0 0;
  max-width: 440px;
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: 17px;
  line-height: 1.5;
  color: #6e6862;
}

.model-panel p {
  color: rgba(255, 255, 255, 0.72);
}

.signal-connector {
  display: flex;
  align-items: center;
  justify-content: center;
}

.signal-connector span {
  width: 42px;
  height: 1px;
  background: rgba(17, 17, 17, 0.26);
  position: relative;
}

.signal-connector span::after {
  content: "";
  position: absolute;
  right: -1px;
  top: 50%;
  width: 8px;
  height: 8px;
  border-top: 1px solid rgba(17, 17, 17, 0.38);
  border-right: 1px solid rgba(17, 17, 17, 0.38);
  transform: translateY(-50%) rotate(45deg);
}

.mini-map {
  position: relative;
  height: 132px;
  margin: 34px 0 34px;
  overflow: hidden;
  background: #f2efea;
  border: 1px solid rgba(17, 17, 17, 0.08);
}

.street-line {
  position: absolute;
  background: rgba(17, 17, 17, 0.12);
}

.street-line-main {
  left: 8%;
  right: 8%;
  top: 50%;
  height: 1px;
}

.street-line-side {
  top: 18%;
  bottom: 18%;
  width: 1px;
}

.street-line-a {
  left: 32%;
}

.street-line-b {
  left: 68%;
}

.map-point {
  position: absolute;
  width: 15px;
  height: 15px;
  border-radius: 4px;
  background: var(--accent, #b44a3c);
  transform: translate(-50%, -50%);
}

.point-a {
  left: 24%;
  top: 50%;
}

.point-b {
  left: 36%;
  top: 50%;
}

.point-c {
  left: 48%;
  top: 50%;
}

.point-d {
  left: 62%;
  top: 50%;
}

.point-e {
  left: 75%;
  top: 50%;
}

.point-f {
  left: 68%;
  top: 32%;
  opacity: 0.42;
}

.signal-badge {
  width: fit-content;
  margin-top: 30px;
  padding: 10px 13px 11px;
  border: 1px solid rgba(255, 255, 255, 0.18);
  background: rgba(255, 255, 255, 0.06);
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.11em;
  text-transform: uppercase;
  color: rgba(255, 255, 255, 0.86);
}

.model-signal-stack {
  margin-top: 34px;
  display: grid;
  gap: 8px;
}

.signal-rank {
  display: grid;
  grid-template-columns: 34px 1fr;
  align-items: center;
  gap: 12px;
  padding: 11px 12px;
  border: 1px solid rgba(255, 255, 255, 0.14);
  color: rgba(255, 255, 255, 0.52);
}

.signal-rank:last-child {
  border-bottom: 1px solid rgba(255, 255, 255, 0.14);
}

.signal-rank span {
  width: 34px;
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 10px;
  letter-spacing: 0.08em;
}

.signal-rank strong {
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: 14px;
  line-height: 1.2;
  font-weight: 600;
}

.signal-rank-main strong {
  font-size: 16px;
}

.signal-rank-main {
  border-color: rgba(180, 74, 60, 0.65);
  background: rgba(180, 74, 60, 0.18);
  color: #ffffff;
}

.supporting-signals {
  margin-top: 22px;
  padding-top: 22px;
  border-top: 1px solid rgba(17, 17, 17, 0.10);
  display: grid;
  grid-template-columns: 220px 1fr;
  gap: 26px;
  align-items: start;
}

.supporting-head span {
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: #9b948d;
}

.signal-list {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.signal-list span {
  min-height: 34px;
  display: inline-flex;
  align-items: center;
  padding: 0 13px;
  border: 1px solid rgba(17, 17, 17, 0.11);
  background: #ffffff;
  font-family: var(--sans, "IBM Plex Sans", sans-serif);
  font-size: 14px;
  line-height: 1;
  color: #5f5a55;
}

.signal-note {
  margin: 18px 0 0;
  font-family: var(--mono, "IBM Plex Mono", monospace);
  font-size: 11px;
  line-height: 1.55;
  color: #9b948d;
}

@media (max-width: 980px) {
  .signal-frame {
    grid-template-columns: 1fr;
    gap: 12px;
  }

  .signal-connector {
    display: none;
  }

  .signal-panel {
    min-height: 300px;
  }

  .model-signal-stack {
    margin-top: 34px;
    gap: 8px;
  }

  .supporting-signals {
    grid-template-columns: 1fr;
    gap: 14px;
  }
}

@media (max-width: 640px) {
  .model-signal {
    margin-top: 34px;
    padding: 24px 0 26px;
  }

  .signal-panel {
    min-height: 0;
    padding: 26px 24px 28px;
  }

  .signal-panel h3 {
    font-size: clamp(30px, 10vw, 44px);
  }

  .mini-map {
    height: 116px;
  }
}
</style>