<script setup>
import { onMounted, ref } from 'vue'
import DeclineMap from './components/DeclineMap.vue'
import CorridorStreetMap from './components/CorridorStreetMap.vue'
import PublicVoices from './components/PublicVoices.vue'
import CorridorGuideMap from './components/CorridorGuideMap.vue'
import RemainingShareChart from './components/RemainingShareChart.vue'
import ConstraintsSummary from './components/ConstraintsSummary.vue'
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
            The same street space became contested.
          </h1>

          <p class="body-l">
            Outdoor dining changed how sidewalks, curbside parking lanes, and street edges were used.
          </p>

          <p class="body-l">
            In 311 complaints, that change appears as blocked movement, physical setup problems, and permit or status conflicts.
          </p>
        </div>

        <div class="fade">
          <StreetConflictDiagram />
        </div>

        <div class="fade">
          <ReportedConflictChart />
        </div>

        <p class="conflict-note fade">
          These records show reported conflicts, not every cause of disappearance.
        </p>
      </div>
    </section>

    <section id="s6">
      <div class="w">
        <div class="s-num fade">LOCAL CONDITIONS</div>

        <h1 class="hed fade">No single factor explains the pattern.</h1>

        <p class="body-l fade">
          The four highlighted streets contain about one fifth of current Manhattan outdoor dining locations.
        </p>

        <div class="fade">
          <RemainingShareChart />
        </div>

        <p class="body-l fade" style="margin-top: 32px;">
          The pattern is shaped by measurable signals and contextual constraints: nearby activity, reported issues, restaurant activity, regulations, costs, and decisions.
        </p>

        <div class="fade">
          <ConstraintsSummary />
        </div>

        <p class="body-l fade s6-note">
          The model only tests measurable signals. It does not capture every constraint.
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

    <section id="s8">
      <div class="w">
        <div class="s-num fade">FRAGMENTS ON THE STREET</div>

        <h1 class="hed fade">Where it still feels present.</h1>

        <p class="body-l fade">
          Outdoor dining did not disappear everywhere. It became concentrated in a few corridors where it still feels visible, active, and part of the street.
        </p>
      </div>

      <div class="s8-wide fade">
        <CorridorGuideMap />
      </div>

      <div class="w">
        <div class="s8-closing fade">
          <p class="body-l">
            Outdoor dining disappeared from many everyday streets.
          </p>

          <p class="body-l">
            But it still appears in specific places.
          </p>

          <p class="body-l s8-last">
            What remains is no longer everywhere.<br>
            It is somewhere.
          </p>
        </div>
      </div>
    </section>
  </main>
</template>