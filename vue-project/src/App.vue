<script setup>
import { onMounted, ref } from 'vue'
import DeclineMap from './components/DeclineMap.vue'
import ComplaintTypeChart from './components/ComplaintTypeChart.vue'
import CorridorStreetMap from './components/CorridorStreetMap.vue'
import PublicVoices from './components/PublicVoices.vue'
import CorridorGuideMap from './components/CorridorGuideMap.vue'
import RemainingShareChart from './components/RemainingShareChart.vue'
import ConstraintsSummary from './components/ConstraintsSummary.vue'
import CorridorStripDiagram from './components/CorridorStripDiagram.vue'
import LandingHero from './components/LandingHero.vue'

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

<!--
S1: Questioning disappearance
S2: Measuring the decline
S3: Loss of citywide coverage
S4: Street level repetition
S5: Reported issues
S6: Measurable local conditions
S7: Rethinking urban presence
S8: Street level ending
-->

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
          But most of that presence has faded from view.
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
            The decline explains why outdoor dining feels less visible. But it does not explain why some places still have it.
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

        <h1 class="hed fade">318 locations still remain.<br>But they are not evenly visible.</h1>

        <p class="body-l fade">
          The remaining locations did not stay everywhere. Some streets still carry visible clusters. Most streets carry little or nothing.
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
            Most remaining locations are scattered, but some streets still hold visible clusters.
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

    <section id="s5">
      <div class="w">
        <div class="s-num fade">REPORTED ISSUES</div>

        <h1 class="hed fade">Outdoor dining creates friction in street space.</h1>

        <p class="body-l fade">
          The same program does not fit every street in the same way. Some locations remain in the roadway. Others remain on the sidewalk.
        </p>

        <p class="body-l fade">
          311 complaints show where that friction became visible, especially around blocked sidewalks, street access, and setup conditions.
        </p>

        <p class="body-l fade">
          The chart does not explain every disappearance. It shows where outdoor dining became contested.
        </p>

        <div class="fade">
          <ComplaintTypeChart />
        </div>
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
          What remains is no longer everywhere. It is concentrated in a few corridors.
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
            Outdoor dining did not disappear from New York.
          </p>

          <p class="body-l">
            It disappeared from most streets and remained concentrated in a smaller set of corridors.
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