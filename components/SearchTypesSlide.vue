<script setup>
import { useNav } from '@slidev/client'
import { computed } from 'vue'

const { clicks } = useNav()

const showDense   = computed(() => clicks.value >= 1)
const showSparse  = computed(() => clicks.value >= 2)
const showHybrid  = computed(() => clicks.value >= 3)
const showQdrant  = computed(() => clicks.value >= 4)
</script>

<template>
  <div class="st-root">

    <div class="st-title">Способы поиска в базе знаний</div>

    <div class="st-grid">

      <!-- Dense -->
      <div class="st-card st-card--dense" :class="{ 'st-visible': showDense }">
        <div class="st-card-name">Dense</div>
        <div class="st-card-sub">по смыслу</div>

        <div class="st-card-body">
          <div class="st-row st-row--good">✓ синонимы, перефраз</div>
          <div class="st-row st-row--bad">✗ точные термины</div>
        </div>

        <div class="st-card-foot">эмбеддинги</div>
      </div>

      <!-- Sparse -->
      <div class="st-card st-card--sparse" :class="{ 'st-visible': showSparse }">
        <div class="st-card-name">Sparse</div>
        <div class="st-card-sub">по словам</div>

        <div class="st-card-body">
          <div class="st-row st-row--good">✓ точные совпадения</div>
          <div class="st-row st-row--good">✓ коды, имена</div>
          <div class="st-row st-row--bad">✗ синонимы</div>
          <div class="st-row st-row--bad">✗ смысл</div>
        </div>

        <div class="st-card-foot">BM25 · TF-IDF</div>
      </div>

      <!-- Hybrid -->
      <div class="st-card st-card--hybrid" :class="{ 'st-visible': showHybrid }">
        <div class="st-card-badge">★ дефолт</div>
        <div class="st-card-name">Hybrid</div>
        <div class="st-card-sub">по смыслу + словам</div>

        <div class="st-card-body">
          <div class="st-row st-row--good">✓ и смысл, и точные слова</div>
          <div class="st-row st-row--good">✓ доменная лексика</div>
          <div class="st-row st-row--good">✓ цитаты, имена</div>
          <div class="st-row st-row--neutral">+ слияние через RRF</div>
        </div>

        <div class="st-card-foot">dense + sparse</div>
      </div>

    </div>

    <!-- Qdrant footer -->
    <div class="st-qdrant" :class="{ 'st-visible': showQdrant }">
      <div class="st-qdrant-icon">🚀</div>
      <div class="st-qdrant-text">
        <span class="st-qdrant-name">Qdrant</span>
        <span class="st-qdrant-dot">·</span>
        <span>нативный hybrid из коробки</span>
        <span class="st-qdrant-dot">·</span>
        <span>Rust, быстрая</span>
        <span class="st-qdrant-dot">·</span>
        <span>self-hosted</span>
      </div>
    </div>

  </div>
</template>

<style scoped>
.st-root {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 22px 28px 14px;
  box-sizing: border-box;
  gap: 14px;
}

.st-title {
  font-size: 1.1em;
  font-weight: 700;
  color: #93c5fd;
  letter-spacing: 0.02em;
  align-self: flex-start;
}

.st-grid {
  flex: 1;
  display: flex;
  align-items: stretch;
  justify-content: center;
  gap: 18px;
  width: 100%;
}

/* Cards */
.st-card {
  position: relative;
  flex: 1;
  max-width: 250px;
  display: flex;
  flex-direction: column;
  border-radius: 12px;
  padding: 14px 16px 12px;
  gap: 8px;
  opacity: 0;
  transform: translateY(10px);
}

.st-card.st-visible {
  animation: st-appear 0.4s ease-out both;
}

@keyframes st-appear {
  from { opacity: 0; transform: translateY(10px); }
  to   { opacity: 1; transform: translateY(0); }
}

.st-card--dense {
  background: rgba(168, 85, 247, 0.10);
  border: 1.5px solid #a855f7;
}

.st-card--sparse {
  background: rgba(245, 158, 11, 0.10);
  border: 1.5px solid #f59e0b;
}

.st-card--hybrid {
  background: rgba(37, 99, 235, 0.16);
  border: 2px solid #3b82f6;
  box-shadow: 0 0 22px rgba(59, 130, 246, 0.26);
}

.st-card-badge {
  position: absolute;
  top: -10px;
  right: 12px;
  background: #3b82f6;
  color: #fff;
  font-size: 0.62em;
  font-weight: 700;
  padding: 3px 10px;
  border-radius: 999px;
  letter-spacing: 0.06em;
}

.st-card-name {
  font-size: 1.1em;
  font-weight: 700;
  color: #e2e8f0;
  letter-spacing: 0.02em;
}

.st-card-sub {
  font-size: 0.72em;
  color: #94a3b8;
  font-style: italic;
  margin-top: -4px;
  margin-bottom: 4px;
}

.st-card-body {
  display: flex;
  flex-direction: column;
  gap: 4px;
  flex: 1;
}

.st-row {
  font-size: 0.74em;
  padding: 3px 0;
  line-height: 1.4;
}

.st-row--good     { color: #6ee7b7; }
.st-row--bad      { color: #fca5a5; }
.st-row--neutral  { color: #94a3b8; font-style: italic; font-size: 0.7em; }

.st-card-foot {
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.62em;
  color: #475569;
  letter-spacing: 0.05em;
  border-top: 1px solid rgba(100, 116, 139, 0.2);
  padding-top: 6px;
  margin-top: 4px;
}

/* Qdrant footer */
.st-qdrant {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 8px 18px;
  background: rgba(37, 99, 235, 0.10);
  border: 1px solid rgba(59, 130, 246, 0.35);
  border-radius: 999px;
  opacity: 0;
  transform: translateY(6px);
  transition: opacity 0.4s, transform 0.4s;
}

.st-qdrant.st-visible {
  opacity: 1;
  transform: translateY(0);
}

.st-qdrant-icon { font-size: 1.1em; }

.st-qdrant-text {
  font-size: 0.78em;
  color: #cbd5e1;
  display: flex;
  align-items: center;
  gap: 8px;
}

.st-qdrant-name {
  color: #93c5fd;
  font-weight: 700;
  letter-spacing: 0.02em;
}

.st-qdrant-dot {
  color: #475569;
}
</style>
