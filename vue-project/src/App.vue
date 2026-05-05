<script setup>
import { onMounted, ref } from 'vue'
import DeclineMap from './components/DeclineMap.vue'
import CorridorStreetMap from './components/CorridorStreetMap.vue'
import PublicVoices from './components/PublicVoices.vue'
import CorridorGuideMap from './components/CorridorGuideMap.vue'
import CorridorStripDiagram from './components/CorridorStripDiagram.vue'
import LandingHero from './components/LandingHero.vue'
import StreetConflictDiagram from './components/StreetConflictDiagram.vue'
import ReportedConflictChart from './components/ReportedConflictChart.vue'

const activeLayer = ref(null)
const activeCorridor = ref('Amsterdam Avenue')

function handleStreetSelected(street) {
  activeCorridor.value = street
}

onMounted(() => {
  const fadeElements = document.querySelectorAll('.fade')

  const observer = new IntersectionObserver((entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        entry.target.classList.add('visible')
      } else {
        entry.target.classList.remove('visible')
      }
    })
  }, { threshold: 0.1 })

  fadeElements.forEach((el) => observer.observe(el))
})
</script>

<template>
  <main>
    <LandingHero />

    <section id="s1b">
      <div class="public-wrap">
        <div class="public-intro fade">
          <div class="s-num">PUBLIC COMMENTS</div>

          <h1 class="public-hed">
            Outdoor dining drew support and pushback.
          </h1>

          <p class="public-dek">
            Some saw public value. Others saw inconvenience.
          </p>
        </div>

        <div class="fade">
          <PublicVoices />
        </div>

        <p class="public-bridge fade">
          But that presence did not last.
        </p>
      </div>
    </section>

    <section id="s2" class="section-decline">
      <div class="decline-grid">
        <div class="decline-copy">
          <div class="s-num fade">NEW YORK OUTDOOR DINING</div>

          <h1 class="hed fade">Most of it disappeared.</h1>

          <div class="decline-stats fade">
            <div class="decline-main">
              <span
                class="decline-from"
                @mouseenter="activeLayer = 'historic'"
                @mouseleave="activeLayer = null"
              >
                4,660
              </span>

              <span class="decline-arrow">→</span>

              <span
                class="decline-to"
                @mouseenter="activeLayer = 'current'"
                @mouseleave="activeLayer = null"
              >
                318
              </span>
            </div>

            <div class="decline-sub">Manhattan outdoor dining locations</div>
            <div class="decline-loss">−93%</div>
          </div>

          <p class="body-l fade">
            Outdoor dining declined sharply after the pandemic emergency ended.
            What remains is part of a smaller permanent program with different rules, costs, and geography.
          </p>
        </div>

        <div class="decline-map-wrap fade">
          <div class="decline-map">
            <DeclineMap :active-layer="activeLayer" />
          </div>

          <div class="decline-legend">
            <div class="legend-row">
              <span class="legend-dot legend-dot-historic"></span>
              <span>Peak (2024)</span>
            </div>

            <div class="legend-row">
              <span class="legend-dot legend-dot-current"></span>
              <span>Current (2026)</span>
            </div>
          </div>
        </div>
      </div>
    </section>

    <section id="s3">
      <div class="w">
        <div class="s-num fade">NOT COMPLETELY GONE</div>

        <h1 class="hed fade">
          318 locations still remain.<br>
          But they are not evenly visible.
        </h1>

        <p class="body-l fade">
          The decline was citywide, but the remainder is uneven. Some streets still carry visible clusters. Most streets carry little or nothing.
        </p>

        <p class="body-l fade">
          To understand why outdoor dining faded from view, I looked at where the remaining locations repeat.
        </p>
      </div>
    </section>

    <section id="s4" class="section-corridor">
      <div class="corridor-grid fade">
        <div class="corridor-copy">
          <div class="s-num fade">WHERE REMAINING LOCATIONS REPEAT</div>

          <h1 class="hed">A few streets carry what remains.</h1>

          <p class="body-l">
            Most remaining locations are scattered. But on some streets, they repeat enough to still feel present.
          </p>

          <div class="corridor-street-list-wrap">
            <CorridorStreetMap @streetSelected="handleStreetSelected" />
          </div>
        </div>

        <div class="corridor-map">
          <CorridorStripDiagram :active-corridor="activeCorridor" />
        </div>
      </div>
    </section>

    <section id="s5" class="section-conflict">
      <div class="conflict-wrap">
        <div class="conflict-intro fade">
          <div class="s-num">CONTESTED SPACE</div>

          <h1 class="hed">
            Outdoor dining turned street space into contested space.
          </h1>

          <p class="body-l">
            In 311 complaints, outdoor dining appears as conflict over shared street space: blocked movement, physical setup problems, and permit or status issues.
          </p>
        </div>

        <div class="conflict-diagram-block fade">
          <StreetConflictDiagram />
        </div>

        <div class="conflict-chart-block fade">
          <ReportedConflictChart />
        </div>

      </div>
    </section>

    <section id="s6" class="section-data-boundary">
      <div class="w s6-inner">
        <div class="s-num fade">WHAT THE DATA CAN SHOW</div>

        <h1 class="hed fade">
          A pattern, not a <br>full explanation.
        </h1>

        <div class="s6-body">
          <p class="body-l fade">
            This project can measure the decline, map current locations, and show where remaining outdoor dining repeats.
          </p>

          <p class="body-l fade">
            It can also show where public comments and 311 complaints made street conflict visible.
          </p>

          <p class="body-l fade">
            But it cannot fully explain each restaurant decision, permit cost, seasonal choice, or block level constraint.
          </p>
        </div>

        <div class="s6-boundary fade">
          <div class="s6-boundary-item">
            <span class="s6-boundary-label">This project shows</span>
            <p>decline, remaining locations, repeated streets, public responses, and reported conflicts</p>
          </div>

          <div class="s6-boundary-item">
            <span class="s6-boundary-label">This project cannot prove</span>
            <p>the full reason each restaurant stayed, removed, or did not renew outdoor dining</p>
          </div>
        </div>

        <p class="s6-closing fade">
          So the project focuses on the pattern left behind.
        </p>
      </div>
    </section>

    <section id="s7">
      <div class="w">
        <div class="s-num fade">UNEVENLY DISTRIBUTED</div>

        <h1 class="hed fade">It disappeared unevenly.</h1>

        <p class="body-l fade">
          Outdoor dining did not vanish from New York. But it disappeared from many everyday streets and remained concentrated in a smaller set of corridors.
        </p>

        <p class="body-l fade">
          A smaller set of streets and neighborhoods now carry most of what remains. Elsewhere, it becomes sparse or disappears entirely.
        </p>
      </div>
    </section>

    <section id="s8" class="section-street-level">
      <div class="w s8-head">
        <div class="s-num fade">STREET LEVEL</div>

        <h1 class="hed fade">What remains appears street by street.</h1>

        <p class="body-l fade">
          Outdoor dining is still visible, but not everywhere.
        </p>
      </div>

      <div class="w-wide fade">
        <CorridorGuideMap />
      </div>
    </section>
  </main>
</template>