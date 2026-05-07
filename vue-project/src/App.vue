<script setup>
import { onMounted, ref } from 'vue'
import DeclineMap from './components/DeclineMap.vue'
import CorridorStreetMap from './components/CorridorStreetMap.vue'
import PublicVoices from './components/PublicVoices.vue'
import CorridorGuideMap from './components/CorridorGuideMap.vue'
import RepetitionProfile from './components/RepetitionProfile.vue'
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

    <!-- ======================== -->
    <!-- S2: DECLINE -->
    <!-- ======================== -->
    <section id="s2" class="section-decline">
      <div class="decline-grid">
        <div class="decline-copy">
          <div class="s-num fade">NEW YORK OUTDOOR DINING</div>

          <h1 class="hed fade">
            Most of it disappeared.
          </h1>

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
            At its peak, outdoor dining spread across Manhattan streets.
            Today, only a small fraction remains.
          </p>

          <p class="body-l fade">
            The loss was not only numerical. It changed where outdoor dining can still be seen.
          </p>

          <p class="body-l fade decline-note">
            Use the map to compare the former footprint with the smaller set of locations that remain.
          </p>
        </div>

        <div class="decline-map-wrap fade">
          <div class="decline-map">
            <DeclineMap :active-layer="activeLayer" />
          </div>

          <div class="decline-legend">
            <div class="legend-row">
              <span class="legend-dot legend-dot-historic"></span>
              <span>Peak program footprint</span>
            </div>

            <div class="legend-row">
              <span class="legend-square legend-square-current"></span>
              <span>Current outdoor dining locations</span>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- ======================== -->
    <!-- S3: VISIBILITY -->
    <!-- ======================== -->
    <section id="s3" class="section-visibility">
      <div class="visibility-wrap">
        <div class="visibility-head fade">
          <div class="s-num">NOT COMPLETELY GONE</div>

          <h1 class="visibility-title">
            318 remain.<br>
            But remaining does not mean visible.
          </h1>

          <p class="visibility-dek">
            Outdoor dining is still present, but it occupies a much smaller part of Manhattan’s street network.
          </p>
        </div>

        <div class="visibility-support fade">
          <div class="support-label">Street coverage</div>

          <div class="support-row">
            <div class="support-copy">
              <span class="support-name">Former footprint</span>
              <span class="support-value">100%</span>
            </div>

            <div class="support-bar former" aria-hidden="true">
              <span></span>
            </div>
          </div>

          <div class="support-row">
            <div class="support-copy">
              <span class="support-name">Current footprint</span>
              <span class="support-value">9.5%</span>
            </div>

            <div class="support-bar current" aria-hidden="true">
              <span></span>
            </div>
          </div>
          <p class="support-note">
            Current outdoor dining covers about one tenth of the former mapped footprint.
          </p>
        </div>

        <p class="visibility-closing fade">
          What remains becomes visible where locations repeat along the same street.
        </p>
      </div>
    </section>

    <!-- ======================== -->
    <!-- S4: CORRIDORS -->
    <!-- ======================== -->
    <section id="s4" class="section-corridor">
      <div class="corridor-grid fade">
        <div class="corridor-copy">
          <div class="s-num fade">WHERE REMAINING LOCATIONS REPEAT</div>

          <h1 class="hed">
            A few streets carry what remains.
          </h1>

          <p class="body-l">
            Most remaining locations are scattered. But on some streets, outdoor dining repeats enough to still feel present.
          </p>

          <p class="body-l">
            These repeated streets turn survival into visibility.
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

    <!-- ======================== -->
    <!-- S5: CONTESTED SPACE -->
    <!-- ======================== -->
    <section id="s5" class="section-conflict">
      <div class="conflict-wrap">
        <div class="conflict-hero fade">
          <div class="s-num">CONTESTED SPACE</div>

          <h1 class="conflict-title">
            Outdoor dining became a fight over shared street space.
          </h1>

          <p class="conflict-dek">
            Public comments show what people argued. 311 complaints show how those conflicts appeared as everyday street problems.
          </p>
        </div>

        <div class="conflict-sources fade">
          <div>
            <span>Public comments</span>
            <p>What people argued</p>
          </div>

          <div>
            <span>311 complaints</span>
            <p>What people reported</p>
          </div>
        </div>

        <div class="quote-stage fade">
          <div class="quote-card">
            <div class="quote-label">Public value</div>

            <p class="quote-text">
              Outdoor dining kept restaurants alive and made streets feel active.
            </p>

            <p class="quote-note">
              Supportive comments framed outdoor dining as recovery, access, street life, and a better use of curb space than parking.
            </p>
          </div>

          <div class="quote-card quote-card-dark">
            <div class="quote-label">Street conflict</div>

            <p class="quote-text">
              Outdoor dining blocked movement, created noise, and turned public space into restaurant space.
            </p>

            <p class="quote-note">
              Critical comments framed outdoor dining as obstruction, sanitation risk, parking loss, noise, and privatized public space.
            </p>
          </div>
        </div>

        <div class="conflict-turn fade">
          <p>
            The argument was not abstract. In 311 complaints, outdoor dining appears through specific problems people reported to the city.
          </p>
        </div>

        <div class="conflict-diagram-block fade">
          <StreetConflictDiagram />
        </div>

        <div class="conflict-chart-block fade">
          <div class="chart-intro">
            <div class="s-num">REPORTED CONFLICTS</div>

            <h2>
              The complaints make the conflict measurable.
            </h2>

            <p>
              311 records show outdoor dining not only as a policy debate, but as a recurring set of reported problems in public space.
            </p>
          </div>

          <ReportedConflictChart />
        </div>

        <div class="regulation-bridge fade">
          <div class="regulation-copy">
            <div class="s-num">REGULATION</div>

            <h2>
              The permanent program made outdoor dining more conditional.
            </h2>

            <p>
              What remained was shaped by new limits on where outdoor dining could be placed, how much space it could take, when roadway seating could operate, and how setups had to be maintained.
            </p>
          </div>

          <div class="regulation-rules">
            <div class="rule-card">
              <span>01</span>
              <p>Sidewalk and roadway seating must fit siting rules and preserve required clearance.</p>
            </div>

            <div class="rule-card">
              <span>02</span>
              <p>Roadway cafes became seasonal, changing permanent sheds into removable setups.</p>
            </div>

            <div class="rule-card">
              <span>03</span>
              <p>Restaurants face added applications, costs, storage, design rules, and compliance burdens.</p>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- ======================== -->
    <!-- S6: MODEL READING -->
    <!-- ======================== -->
    <section id="s6">
      <div class="w">
        <div class="s-num fade">MODEL READING</div>

        <h1 class="hed fade">
          The model reads survival as a pattern of repetition.
        </h1>

        <p class="body-l fade">
          I used a classification model to compare locations that remained with locations that disappeared.
        </p>

        <p class="body-l fade">
          The clearest signal was not one single cause. It was nearby repetition, especially where outdoor dining repeats along the same street.
        </p>

        <RepetitionProfile class="fade" />

        <p class="body-l fade s6-note">
          This does not explain why each restaurant stayed. It shows that the remaining pattern becomes more visible where outdoor dining repeats locally.
        </p>
      </div>
    </section>

    <!-- ======================== -->
    <!-- S7: CONCLUSION -->
    <!-- ======================== -->
    <section id="s7">
      <div class="w">
        <div class="s-num fade">WHAT CHANGED</div>

        <h1 class="hed fade">
          Outdoor dining did not simply disappear.
          It was reduced, concentrated, and contested.
        </h1>

        <p class="body-l fade">
          Outdoor dining still exists in Manhattan, but it no longer works as a broad street level condition.
        </p>

        <p class="body-l fade">
          What remains is carried by fewer corridors, shaped by new rules, and negotiated through public conflict.
        </p>

        <p class="body-l fade">
          This is why outdoor dining can still be present in the city, while feeling absent from everyday streets.
        </p>
      </div>
    </section>

    <!-- ======================== -->
    <!-- S8: STREET LEVEL -->
    <!-- ======================== -->
    <section id="s8" class="section-street-level">
      <div class="w s8-head">
        <div class="s-num fade">STREET LEVEL</div>

        <h1 class="hed fade">
          What remains appears street by street.
        </h1>

        <p class="body-l fade">
          Outdoor dining is still visible, but not everywhere.
        </p>

        <p class="body-l fade">
          The remaining pattern is easiest to see from the street, where repeated locations make some corridors feel active while others feel empty.
        </p>
      </div>

      <div class="w-wide fade">
        <CorridorGuideMap />
      </div>
    </section>
  </main>
</template>