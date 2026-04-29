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
    <section id="s1">
      <div class="s1-images fade">
        <div class="s1-image s1-image-left">
          <img src="/images/s1-vibrant.jpg" alt="Vibrant outdoor dining during pandemic" />
          <div class="s1-caption">Vibrant</div>
        </div>

        <div class="s1-image s1-image-right">
          <img src="/images/s1-shed.jpg" alt="Empty or contested outdoor dining structures" />
          <div class="s1-caption">Contested</div>
        </div>
      </div>

      <div class="w">
        <h1 class="hed hed-main fade">Why does outdoor dining feel gone?</h1>

        <p class="body-l fade">
          It didn’t disappear. It became uneven.
        </p>
      </div>
    </section>

    <section id="s1b">
      <div class="w">
        <p class="body-l fade">
          Outdoor dining was not simply accepted or rejected. It produced both public value and public conflict.
        </p>

        <div class="fade">
          <PublicVoices />
        </div>

        <p class="body-l fade">
          But despite this, most of it is now gone. What remains is not evenly distributed.
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

        <h1 class="hed fade">But 318 locations are still there.<br>So where are they?</h1>

        <p class="body-l fade">
          The remaining locations did not stay everywhere. They concentrated.
          Some streets retained visible clusters. Most streets retained little or nothing.
        </p>

        <p class="body-l fade">
          The question is not only how many remain. It is where they remain.
        </p>
      </div>
    </section>

    <section id="s4" class="section-corridor">
      <div class="corridor-grid fade">
        <div class="corridor-copy">
          <div class="s-num fade">WHERE IT STILL APPEARS</div>

          <h1 class="hed">What remains repeats along a few streets.</h1>

          <p class="body-l">
            Most locations are scattered. What stands out is the small number of streets where outdoor dining still repeats along the same corridor.
          </p>

          <CorridorStreetMap @streetSelected="handleStreetSelected" />
        </div>

        <div class="corridor-map">
          <CorridorStripDiagram :active-corridor="activeCorridor" />
        </div>
      </div>
    </section>

    <section id="s5">
      <div class="w">
        <div class="s-num fade">REPORTED ISSUES</div>

        <h1 class="hed fade">Outdoor dining doesn't just occupy space.<br>It competes for it.</h1>

        <p class="body-l fade">
          This is not just about dining. It is about how public space is used.
          When restaurants place tables on sidewalks or in the street, they take space that pedestrians, cyclists, and delivery vehicles also rely on.
        </p>

        <p class="body-l fade">
          311 complaints add another layer to the story.
        </p>

        <p class="body-l fade">
          These records do not show where outdoor dining remains.
          They show where people reported problems around outdoor dining.
        </p>

        <div class="fade">
          <ComplaintTypeChart />
        </div>
      </div>
    </section>

    <section id="s6">
      <div class="w">
        <div class="s-num fade">LOCAL CONDITIONS</div>

        <h1 class="hed fade">What remains is not random.</h1>

        <p class="body-l fade">
          Outdoor dining did not disappear evenly. It remained at a higher rate along repeated corridor streets.
        </p>

        <div class="fade">
          <RemainingShareChart />
        </div>

        <p class="body-l fade" style="margin-top: 32px;">
          But this chart does not explain the whole pattern. It only shows that the decline was uneven.
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

        <h1 class="hed fade">It did not disappear.<br>It became uneven.</h1>

        <p class="body-l fade">
          Outdoor dining no longer feels like a normal part of most streets, because most of it is no longer visible.
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
            Outdoor dining did not disappear from New York.
          </p>

          <p class="body-l">
            It disappeared from the everyday map of the city.
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

<style scoped>
.s8-wide {
  width: min(1280px, calc(100vw - 120px));
  margin: 40px auto 0;
}

.s8-closing {
  margin-top: 48px;
}

.s8-last {
  font-weight: 500;
  color: var(--ink);
  margin-top: 8px;
}
</style>