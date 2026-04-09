<template>
  <div class="cards">
    <div class="card">
      <div class="num green">141</div>
      <div class="label">High access, low friction</div>
    </div>

    <div class="card">
      <div class="num red">25</div>
      <div class="label">Low access, high friction</div>
    </div>
  </div>
</template>

<style scoped>
.cards {
  display: grid;
  grid-template-columns: 1fr 1fr;
  margin-top: 28px;
  border: 1px solid #ddd;
}

.card {
  padding: 24px;
}

.num {
  font-size: 42px;
  font-weight: 700;
}

.green {
  color: #3ccf7a;
}

.red {
  color: #ff5a5a;
}

.label {
  font-size: 12px;
  color: #999;
}
</style>
