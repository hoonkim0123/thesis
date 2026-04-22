<script setup>
import { onMounted, ref } from 'vue'
import DeclineMap from './components/DeclineMap.vue'
import ComplaintTypeChart from './components/ComplaintTypeChart.vue'
import StatCards from './components/StatCards.vue'
import CorridorStreetMap from './components/CorridorStreetMap.vue'
import CorridorMap from './components/CorridorMap.vue'
import PublicVoices from './components/PublicVoices.vue'
import CorridorGuideMap from './components/CorridorGuideMap.vue'

const corridorMapRef = ref(null)

function handleStreetSelected(street) {
  // Pass the selected street to CorridorMap
  if (corridorMapRef.value) {
    corridorMapRef.value.highlightStreet(street)
  }
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
          Outdoor dining was not simply accepted or rejected. It produced both public value and public friction.
        </p>

        <div class="fade">
          <PublicVoices />
        </div>

        <p class="body-l fade">
          But despite this, most of it is now gone. What remains is not evenly distributed.
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
            <div class="decline-main">
              <span class="decline-from muted">4,660</span>
              <span class="decline-arrow">→</span>
              <span class="decline-to">318</span>
            </div>
            <div class="decline-sub">Peak → Current</div>
            <div class="decline-loss">−93%</div>
          </div>

          <p class="body-l fade">
            Outdoor dining declined sharply after the pandemic emergency ended. What remains is not random. It reflects a different program, with different rules, costs, and geography.
          </p>
        </div>

        <div class="decline-map-wrap fade">
          <div class="decline-map">
            <DeclineMap />
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
        <div class="s-num fade">03</div>
        <h1 class="hed fade">But 318 locations are still there.<br>So where are they?</h1>
        
        <p class="body-l fade">
          The remaining locations didn't stay everywhere. They concentrated. Some streets retained visible clusters. Most streets retained little or nothing.
        </p>

        <p class="body-l fade">
          The question isn't just how many survived. It's where.
        </p>
      </div>
    </section>

    <!-- ======================== -->
    <!-- S4: Street corridors    -->
    <!-- ======================== -->
    <section id="s4" class="section-corridor">
      <div class="corridor-grid fade">
        <div class="corridor-copy">
          <div class="s-num">04</div>
          <h1 class="hed">What remains clusters on just a few streets.</h1>
          
          <p class="body-l">
            Most locations are scattered. What stands out is the small number of streets where outdoor dining still repeats.
          </p>

          <CorridorStreetMap @streetSelected="handleStreetSelected" />
        </div>

        <div class="corridor-map">
          <CorridorMap ref="corridorMapRef" />
        </div>
      </div>
    </section>

    <!-- ======================== -->
    <!-- S5: WHY                  -->
    <!-- ======================== -->
    <section id="s5">
      <div class="w">
        <div class="s-num fade">05</div>
        <h1 class="hed fade">Outdoor dining doesn't just occupy space.<br>It competes for it.</h1>
        
        <p class="body-l fade">
          This is not just about dining. It is about how public space is used. When restaurants place tables on sidewalks or in the street, they take space that pedestrians, cyclists, and delivery vehicles also rely on.
        </p>

        <p class="body-l fade">
          311 complaints show where that conflict becomes visible.
        </p>

        <div class="fade">
          <ComplaintTypeChart />
        </div>
      </div>
    </section>

    <!-- ======================== -->
    <!-- S6: CONDITIONS           -->
    <!-- ======================== -->
    <section id="s6">
      <div class="w">
        <div class="s-num fade">06</div>
        <h1 class="hed fade">Not all places can support it.</h1>
        
        <p class="body-l fade">
          The remaining locations do not share the same conditions. Some cluster in more connected parts of the city, while others remain more isolated or contested.
        </p>

        <p class="body-l fade">
          The same system produces very different conditions depending on location. In some areas, outdoor dining feels visible and accessible. In others, it is rare or absent.
        </p>

        <div class="fade">
          <StatCards />
        </div>

        <div class="fade">
        </div>
      </div>
    </section>

    <!-- ======================== -->
    <!-- S7: REFRAME              -->
    <!-- ======================== -->
    <section id="s7">
      <div class="w">
        <div class="s-num fade">07</div>
        <h1 class="hed fade">It didn't disappear.<br>It became uneven.</h1>
        
        <p class="body-l fade">
          Outdoor dining no longer spreads across the city. It concentrates where access is higher and friction is lower.
        </p>

        <p class="body-l fade">
          A smaller set of streets now carries most of what remains. Elsewhere, it becomes sparse or disappears entirely.
        </p>

        <div class="fade" style="border-top: 1px solid var(--rule); border-bottom: 1px solid var(--rule); padding: 32px 0; margin: 40px 0;">
          <p class="body-l" style="margin-bottom: 16px; font-family: var(--sans); font-size: 20px; font-weight: 600; color: var(--ink); line-height: 1.6; letter-spacing: -0.01em;">
            <em>Outdoor dining is still there. But it no longer feels like it is.</em>
          </p>
          <p class="body-l" style="margin-bottom: 14px;">
            And yet, it still exists — but only in certain parts of the city. If you want to experience it, you now have to know where to go.
          </p>
        </div>
      </div>
    </section>

    <!-- ======================== -->
    <!-- S8: WHERE IT STILL IS   -->
    <!-- ======================== -->
    <section id="s8">
      <div class="w">
        <div class="s-num fade">08</div>
        <h1 class="hed fade">Where to find it now.</h1>

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
            What remains is not random. It survives where cost, regulation, and local street conditions still allow it.
          </p>
          <p class="body-l s8-last">
            What remains is fragile, but patterned. Outdoor dining still exists where the city can still support it.
          </p>
        </div>
      </div>
    </section>
  </main>
</template>\