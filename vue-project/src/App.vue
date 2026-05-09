<script setup>
import { onMounted, ref } from 'vue'
import DeclineMap from './components/DeclineMap.vue'
import CorridorStreetMap from './components/CorridorStreetMap.vue'
import CorridorGuideMap from './components/CorridorGuideMap.vue'
import RepetitionProfile from './components/RepetitionProfile.vue'
import CorridorStripDiagram from './components/CorridorStripDiagram.vue'
import LandingHero from './components/LandingHero.vue'
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
              <span class="decline-from">
                4,660
              </span>

              <span class="decline-arrow">→</span>

              <span class="decline-to">
                318
              </span>
            </div>

            <div class="decline-sub">Manhattan outdoor dining locations</div>
            <div class="decline-loss">−93% drop</div>
          </div>

          <p class="body-l fade">
            At its peak, outdoor dining spread across Manhattan streets. Today, only 318 locations remain, a 93 percent drop.
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
    <!-- S5: MODEL READING -->
    <!-- ======================== -->
    <section id="s5" class="section-model">
      <div class="w">
        <div class="s-num fade">MODEL READING</div>

        <h1 class="hed fade">
          The model reads where repetition remains.
        </h1>

        <p class="body-l fade">
          After seeing this pattern in the map, I wanted to see whether the model would read it too.
        </p>

        <p class="body-l fade">
          The model does not prove why each restaurant stayed or left. It shows that the strongest signal was nearby repetition on the same street.
        </p>

        <RepetitionProfile class="fade" />

        <p class="body-l fade s6-note">
          Random Forest feature importance. Exploratory, not causal.
        </p>
      </div>
    </section>

    <!-- ======================== -->
    <!-- S6: CONTESTED SPACE -->
    <!-- ======================== -->
    <section id="s6" class="section-contested">
      <div class="w contested-wrap">
        <div class="s-num fade">CONTESTED SPACE</div>

        <h1 class="hed fade">
          Outdoor dining has also become a conflict over shared street space.
        </h1>

        <p class="body-l fade contested-intro">
          Spatial repetition explains where outdoor dining remains visible.
          But it does not explain the full public debate around it.
        </p>

        <p class="body-l fade contested-intro">
          Public comments show what people argued. 311 complaints show what people reported.
        </p>

        <!-- Editorial public comments block -->
        <div class="comment-editorial fade">
          <div class="editorial-subhead">Two readings of the same street</div>

          <div class="editorial-stagger">
            <div class="stagger-quote stagger-quote-left">
              <div class="quote-type">SUPPORTIVE</div>
              <blockquote>
                “They have brought both life and creativity to our streets.”
              </blockquote>
            </div>

            <div class="stagger-quote stagger-quote-right">
              <div class="quote-type">OPPOSED</div>
              <blockquote>
                “The sidewalks are for the movement of pedestrians, not seating for restaurants.”
              </blockquote>
            </div>
          </div>

          <p class="editorial-note">
            Public comments show the argument around outdoor dining: for some, street life and recovery; for others, obstruction and loss of public space.
          </p>
        </div>

        <!-- 311 complaints section -->
        <div class="complaints-block fade">

          <h2 class="subhed">
            The complaints make the conflict measurable.
          </h2>

          <p class="body-l complaints-intro">
            311 records show how those arguments appeared as recurring street-level problems.
          </p>

          <ReportedConflictChart />
        </div>
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