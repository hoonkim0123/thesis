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
            At its peak, outdoor dining spread across Manhattan streets. Today, only a small fraction remains.
          </p>

          <p class="body-l fade decline-note">
            The map compares the emergency program footprint with the smaller set of locations that remain today.
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
            But scattered locations are easy to miss.
          </h1>

          <p class="visibility-dek">
            A single setup can disappear into the street. Outdoor dining becomes visible when locations repeat along the same street.
          </p>
        </div>

        <div class="visibility-pattern fade">
          <div class="pattern-label">Street visibility</div>

          <div class="pattern-row">
            <div class="pattern-copy">
              <span class="pattern-name">Isolated</span>
            </div>

            <div class="dot-field isolated-field" aria-hidden="true">
              <span class="dot dot-current dot-a"></span>
            </div>
          </div>

          <div class="pattern-row">
            <div class="pattern-copy">
              <span class="pattern-name">Repeated</span>
            </div>

            <div class="dot-field repeated-field" aria-hidden="true">
              <span class="dot dot-current dot-a"></span>
              <span class="dot dot-current dot-b"></span>
              <span class="dot dot-current dot-c"></span>
              <span class="dot dot-current dot-d"></span>
              <span class="dot dot-current dot-e"></span>
            </div>
          </div>

          <p class="pattern-note">
            Visibility depends on repetition, not just presence.
          </p>
        </div>
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
            A few streets make it visible.
          </h1>

          <p class="body-l">
            Only a few streets still show repeated setups.
          </p>

          <p class="body-l">
            On these streets, visibility comes from locations appearing close enough to be noticed together.
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
          The model reads the same street pattern.
        </h1>

        <p class="body-l fade">
          I used the model to test whether the same pattern appeared in the data.
        </p>

        <p class="body-l fade">
          It does not prove why each restaurant stayed or left. But among the features tested, same-street repetition was the strongest signal.
        </p>

        <RepetitionProfile class="fade" />

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
          Repetition explains where outdoor dining remains visible. But public comments and 311 complaints show why that visibility became contested.
        </p>

        <!-- Editorial public comments block -->
        <div class="comment-editorial fade">
          <div class="editorial-subhead">The same street can be read two ways.</div>

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
        </div>

        <!-- 311 complaints section -->
        <div class="complaints-block fade">

          <h2 class="subhed">
            The complaints make the conflict measurable.
          </h2>

          <p class="body-l complaints-intro">
            311 records show how the conflict appeared as recurring problems on the street.
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
          Outdoor dining did not simply disappear. It was reduced, concentrated, and contested.
        </h1>

        <p class="body-l fade">
          The city still has outdoor dining, but it no longer appears as a broad street level condition.
        </p>

        <p class="body-l fade">
          What remains is shaped by fewer streets, new rules, and public conflict.
        </p>

        <p class="body-l fade">
          To see that change, the project returns to the street.
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
          At street level, outdoor dining is still visible, but not everywhere. Repeated locations make some corridors feel present while other streets feel absent.
        </p>
      </div>

      <div class="w-wide fade">
        <CorridorGuideMap />
      </div>
    </section>
  </main>
</template>