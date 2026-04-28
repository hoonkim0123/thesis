<script setup>
import { onMounted, ref } from 'vue'
import DeclineMap from './components/DeclineMap.vue'
import ComplaintTypeChart from './components/ComplaintTypeChart.vue'
import StatCards from './components/StatCards.vue'
import CorridorStreetMap from './components/CorridorStreetMap.vue'
import CorridorMap from './components/CorridorMap.vue'
import PublicVoices from './components/PublicVoices.vue'
import CorridorGuideMap from './components/CorridorGuideMap.vue'
import ModelFeatureChart from './components/ModelFeatureChart.vue'

const corridorMapRef = ref(null)
const activeLayer = ref(null)
const hoveredCorridor = ref(null)

function handleStreetSelected(street) {
  // Pass the selected street to CorridorMap
  if (corridorMapRef.value) {
    corridorMapRef.value.highlightStreet(street)
  }
}

function handlePointHover(corridor) {
  hoveredCorridor.value = corridor
}

function handlePointLeave() {
  hoveredCorridor.value = null
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
S4: Emergence of street-level patterns  
S5: Value and friction  
S6: Conditions of survival  
S7: Rethinking urban presence

DATA:
- thesis_points_final_v2.geojson
- historic_manhattan_clean.geojson
- thesis_summary_v2.json
- thesis_complaint_descriptors.csv
- MTA subway stations
- corridor_points_labeled.geojson
- corridor_streets.geojson
- corridor_labels.geojson
-->

<template>
  <main>
    <!-- ======================== -->
    <!-- S1: HOOK                 -->
    <!-- ======================== -->
    <section id="s1">
      <!-- S1 Images -->
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

    <!-- ======================== -->
    <!-- S1B: PUBLIC VOICES       -->
    <!-- ======================== -->
    <section id="s1b">
      <div class="w">
        <p class="body-l fade">
          Some saw outdoor dining as public life.
          Others saw it as noise, obstruction, and conflict.
        </p>

        <div class="fade">
          <PublicVoices />
        </div>

        <p class="body-l fade">
          It created value and friction at the same time.
        </p>
      </div>
    </section>

    <!-- ======================== -->
    <!-- S2: DECLINE              -->
    <!-- ======================== -->
    <section id="s2" class="section-decline">
      <div class="decline-grid">
        <div class="decline-copy">
          <div class="s-num fade">NEW YORK OUTDOOR DINING</div>
          <h1 class="hed fade">Most of it disappeared.</h1>

          <div class="decline-stats fade">
            <div class="decline-number-row">
              <span
                class="stat-old"
                @mouseenter="activeLayer = 'historic'"
                @mouseleave="activeLayer = null"
              >
                4,660
              </span>
              <span class="stat-arrow">→</span>
              <span
                class="stat-current"
                @mouseenter="activeLayer = 'current'"
                @mouseleave="activeLayer = null"
              >
                318
              </span>
            </div>

            <div class="stat-label-row">
              <span>PEAK (2024)</span>
              <span></span>
              <span>CURRENT (2026)</span>
            </div>

            <div class="stat-drop">−93%</div>
          </div>

          <p class="body-l fade">
            Outdoor dining declined sharply after the pandemic.<br>
            What remains is shaped by new rules, costs, and geography.
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

    <!-- ======================== -->
    <!-- S3: TRANSITION           -->
    <!-- ======================== -->
    <section id="s3">
      <div class="w">
        <div class="s-num fade">NOT COMPLETELY GONE</div>
        <h1 class="hed fade">But 318 locations are still there.<br>So where are they?</h1>
        
        <p class="body-l fade">
          It did not stay everywhere.<br>
          It concentrated.
        </p>

        <p class="body-l fade text-break">
          Some streets kept outdoor dining visible.
          Most streets did not.
        </p>
      </div>
    </section>

    <!-- ======================== -->
    <!-- S4: Street corridors    -->
    <!-- ======================== -->
    <section id="s4" class="section-corridor">
      <div class="corridor-grid fade">
        <div class="corridor-copy">
          <div class="s-num fade">WHERE IT STILL APPEARS</div>
          <h1 class="hed fade">The pattern repeats along certain streets.</h1>
          
          <p class="body-l fade">
            Outdoor dining is no longer spread evenly across the city.
            It appears in repeated clusters along specific streets.
          </p>

          <div class="fade">
            <CorridorStreetMap @streetSelected="handleStreetSelected" :hovered-corridor="hoveredCorridor" />
          </div>
        </div>

        <div class="corridor-map">
          <CorridorMap ref="corridorMapRef" @pointHover="handlePointHover" @pointLeave="handlePointLeave" />
        </div>
      </div>
    </section>

    <!-- ======================== -->
    <!-- S5: WHY                  -->
    <!-- ======================== -->
    <section id="s5">
      <div class="w">
        <div class="s-num fade">REPORTED ISSUES</div>
        <h1 class="hed fade">Where conflict appears.</h1>
        
        <p class="body-l fade">
          Outdoor dining competes for public space.
        </p>

        <p class="body-l fade">
          311 data shows where these issues appeared most:
          blocked sidewalks, setup conditions, and access issues.
        </p>

        <div class="fade">
          <ComplaintTypeChart />
        </div>

        <p class="body-l fade">
          This is where conflict becomes visible.
        </p>
      </div>
    </section>

    <!-- ======================== -->
    <!-- S6: CONDITIONS           -->
    <!-- ======================== -->
    <section id="s6">
      <div class="w">
        <div class="s-num fade">CONDITIONS AROUND PRESENCE</div>
        <h1 class="hed fade">Survival is not random.</h1>

        <p class="body-l fade">
          But does this actually shape what remains?
        </p>

        <p class="body-l fade text-break">
          I used a simple model to compare locations that remained with those that disappeared.
        </p>

        <div class="fade">
          <ModelFeatureChart />
        </div>

        <p class="body-l fade text-break">
          No single factor determines survival.
          What remains reflects overlapping local conditions.
        </p>

        <p class="body-l fade">
          And this is why the pattern is uneven.
        </p>
      </div>
    </section>

    <!-- ======================== -->
    <!-- S7: REFRAME              -->
    <!-- ======================== -->
    <section id="s7">
      <div class="w">
        <div class="s-num fade">UNEVENLY DISTRIBUTED</div>
        <h1 class="hed fade">It did not disappear.<br>It became uneven.</h1>
        
        <p class="body-l fade">
          Outdoor dining no longer feels like a normal part of most streets,
          because most of it is no longer visible.
        </p>
      </div>
    </section>

    <!-- ======================== -->
    <!-- S8: WHERE IT STILL IS   -->
    <!-- ======================== -->
    <section id="s8">
      <div class="w">
        <div class="s-num fade">FRAGMENTS ON THE STREET</div>
        <h1 class="hed fade">Where it still feels present.</h1>

        <p class="body-l fade">
          It is still there.<br>
          But only in certain streets.
        </p>

        <p class="body-l fade text-break">
          That is where it still feels present.
        </p>
      </div>

      <div class="s8-wide fade">
        <CorridorGuideMap />
      </div>

      <div class="w">
        <div class="s8-closing fade">
          <p class="body-l fade">
            So why does outdoor dining feel gone?
          </p>
          <p class="body-l s8-last fade">
            Because what remains is no longer everywhere.
            It is somewhere.
          </p>
        </div>
      </div>
    </section>
  </main>
</template>\