<script setup>
import { onMounted } from 'vue'
import { CASES } from './data/cases'

const cases = CASES
const getCaseClass = (rate) => {
  if (rate >= 0.15) return 'high'
  if (rate > 0) return 'low'
  return 'none'
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
    <!-- Section 1 -->
    <section id="s1">
      <div class="w">
        <div class="s-tag fade" data-n="1">Overview</div>
        <h1 class="s-hed fade">Where did outdoor dining survive?</h1>
        <p class="s-sub fade">After the pandemic, thousands of temporary dining structures disappeared. But not evenly.</p>
        <div class="placeholder-map fade">Map: All locations (2020-2024)</div>
      </div>
    </section>

    <!-- Section 2 -->
    <section id="s2">
      <div class="w">
        <div class="s-tag fade" data-n="2">Peak</div>
        <h1 class="s-hed fade">They were <em>everywhere.</em></h1>
        <p class="s-sub fade">During COVID-19, New York City issued over 10,000 outdoor dining permits citywide. On every block, restaurants built structures on sidewalks and streets. For two years, the city looked like this.</p>
        <div class="placeholder-map fade">Map: Disappeared vs. Survived</div>
      </div>
    </section>

    <!-- Section 3 -->
    <section id="s3">
      <div class="w">
        <div class="s-tag fade" data-n="3">Today</div>
        <h1 class="s-hed fade">Most have <em>disappeared.</em></h1>
        <p class="s-sub fade">When emergency policies ended, the city introduced a permanent program with stricter requirements. The vast majority of structures were removed. In Manhattan, 93.5% of all pandemic-era dining locations are now gone.</p>
        <div class="placeholder-map fade">Map: Survivors Only</div>
      </div>
    </section>

    <!-- Section 4 -->
    <section id="s4">
      <div class="w">
        <div class="s-tag fade" data-n="4">Analysis</div>
        <h1 class="s-hed fade">Streets with <em>different characteristics</em> had different outcomes.</h1>
        <p class="s-sub fade">Three corridors. All in Manhattan. All active during the pandemic. Today: one kept it, two lost almost everything.</p>
    
        <div class="case-cards fade">
          <div v-for="(caseData, key) in cases" :key="key" :class="['case-card', getCaseClass(caseData.survival_rate)]">
            <div class="case-card-header">
              <h3>{{ caseData.name }}</h3>
              <div class="rate">{{ (caseData.survival_rate * 100).toFixed(1) }}%</div>
              <div class="detail">{{ caseData.survived_pts }}/{{ caseData.total_pts }} permits survived</div>
            </div>
            <div class="case-map">Map: {{ caseData.name }}</div>
          </div>
        </div>
      </div>
    </section>

<!-- Section 5 -->
<section id="s5">
  <div class="w">
    <div class="s-tag fade" data-n="5">Street Friction</div>
    <h1 class="s-hed fade">Different streets experience <em>different kinds of friction.</em></h1>
    <p class="s-sub fade">Mulberry has the most complaints per foot of any corridor — yet the highest survival. This suggests that not all complaints are equal.</p>
    

    
    <div class="placeholder-chart fade">Chart: Complaint breakdown by type</div>
  </div>
</section>
  </main>
</template>

<style scoped></style>
