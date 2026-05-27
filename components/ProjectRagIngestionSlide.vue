<script setup>
import { useNav } from '@slidev/client'
import { computed } from 'vue'

const { clicks } = useNav()

const showChunk  = computed(() => clicks.value >= 1)
const showEmbed  = computed(() => clicks.value >= 2)
const showQdrant = computed(() => clicks.value >= 3)
const showGraph  = computed(() => clicks.value >= 4)
</script>

<template>
  <div class="pri-root">

    <div class="pri-title">Как заметки попадают в RAG</div>

    <div class="pri-pipeline">

      <!-- Source -->
      <div class="pri-step">
        <div class="pri-step-icon">📁</div>
        <div class="pri-step-name">Markdown<br/>файлы</div>
        <div class="pri-step-meta">watcher · 5 мин</div>
      </div>

      <div class="pri-arrow pri-arrow--on">→</div>

      <!-- Chunker -->
      <div class="pri-step" :class="{ 'pri-on': showChunk }">
        <div class="pri-step-icon">✂️</div>
        <div class="pri-step-name">Chunker</div>
        <div class="pri-step-meta">режем по H1/H2<br/>≤3200 символов</div>
      </div>

      <div class="pri-arrow" :class="{ 'pri-arrow--on': showEmbed }">→</div>

      <!-- Embeddings -->
      <div class="pri-step" :class="{ 'pri-on': showEmbed }">
        <div class="pri-step-icon">🔢</div>
        <div class="pri-step-name">BGE-M3</div>
        <div class="pri-step-meta">dense 1024d<br/>+ sparse · GPU</div>
      </div>

      <!-- Branching arrows + dual sinks -->
      <div class="pri-branch">
        <svg class="pri-branch-svg" viewBox="0 0 80 160">
          <defs>
            <marker id="pri-arr" markerWidth="6" markerHeight="6" refX="5" refY="3" orient="auto">
              <polygon points="0,0.5 0,5.5 6,3" fill="#3b82f6"/>
            </marker>
            <marker id="pri-arr-p" markerWidth="6" markerHeight="6" refX="5" refY="3" orient="auto">
              <polygon points="0,0.5 0,5.5 6,3" fill="#a855f7"/>
            </marker>
          </defs>
          <!-- to qdrant (top) -->
          <path v-if="showQdrant" class="pri-line"
            d="M2,80 C30,80 30,40 70,40"
            stroke="#3b82f6" marker-end="url(#pri-arr)"/>
          <!-- to graph (bottom) -->
          <path v-if="showGraph" class="pri-line"
            d="M2,80 C30,80 30,120 70,120"
            stroke="#a855f7" marker-end="url(#pri-arr-p)"/>
        </svg>

        <div class="pri-sinks">
          <div class="pri-sink pri-sink--blue" :class="{ 'pri-on': showQdrant }">
            <div class="pri-sink-icon">🗄️</div>
            <div class="pri-sink-name">Qdrant</div>
            <div class="pri-sink-meta">векторы + метаданные<br/>(дата, теги, проект)</div>
          </div>
          <div class="pri-sink pri-sink--purple" :class="{ 'pri-on': showGraph }">
            <div class="pri-sink-icon">🕸️</div>
            <div class="pri-sink-name">SQLite Graph</div>
            <div class="pri-sink-meta">LLM извлекает<br/>сущности и связи</div>
          </div>
        </div>
      </div>

    </div>

    <div class="pri-foot">~60 файлов → 502 чанка в Qdrant + граф сущностей в SQLite</div>

  </div>
</template>

<style scoped>
.pri-root {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 24px 28px 14px;
  box-sizing: border-box;
  gap: 18px;
}

.pri-title {
  font-size: 1.1em;
  font-weight: 700;
  color: #93c5fd;
  letter-spacing: 0.02em;
  align-self: flex-start;
}

.pri-pipeline {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  width: 100%;
}

/* Generic step */
.pri-step {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  padding: 10px 14px;
  border-radius: 10px;
  background: rgba(71, 85, 105, 0.12);
  border: 1.5px solid #475569;
  min-width: 110px;
  opacity: 0.35;
  transition: opacity 0.35s, border-color 0.35s, background 0.35s;
}

.pri-step.pri-on,
.pri-step:first-of-type {
  opacity: 1;
  border-color: #3b82f6;
  background: rgba(37, 99, 235, 0.15);
}

.pri-step-icon {
  font-size: 1.4em;
  line-height: 1;
}

.pri-step-name {
  font-weight: 700;
  font-size: 0.82em;
  color: #cbd5e1;
  text-align: center;
  line-height: 1.2;
}

.pri-step-meta {
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.6em;
  color: #64748b;
  text-align: center;
  line-height: 1.4;
}

.pri-arrow {
  font-size: 1.2em;
  color: #334155;
  font-weight: 700;
  transition: color 0.35s;
}

.pri-arrow.pri-arrow--on { color: #3b82f6; }

/* Branching */
.pri-branch {
  display: flex;
  align-items: center;
  gap: 0;
  position: relative;
}

.pri-branch-svg {
  width: 60px;
  height: 160px;
  flex-shrink: 0;
}

.pri-line {
  fill: none;
  stroke-width: 1.8;
  stroke-linecap: round;
  stroke-dasharray: 1;
  stroke-dashoffset: 1;
  pathLength: 1;
  animation: pri-draw 0.5s ease-out both;
}

@keyframes pri-draw {
  to { stroke-dashoffset: 0; }
}

.pri-sinks {
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.pri-sink {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
  padding: 8px 14px;
  border-radius: 10px;
  min-width: 150px;
  opacity: 0;
  transform: translateX(8px);
  transition: opacity 0.35s, transform 0.35s;
}

.pri-sink.pri-on {
  opacity: 1;
  transform: translateX(0);
}

.pri-sink--blue {
  background: rgba(37, 99, 235, 0.16);
  border: 1.5px solid #3b82f6;
  box-shadow: 0 0 14px rgba(59, 130, 246, 0.20);
}

.pri-sink--purple {
  background: rgba(168, 85, 247, 0.12);
  border: 1.5px solid #a855f7;
}

.pri-sink-icon {
  font-size: 1.3em;
  line-height: 1;
}

.pri-sink-name {
  font-weight: 700;
  font-size: 0.85em;
  color: #e2e8f0;
}

.pri-sink-meta {
  font-size: 0.62em;
  color: #94a3b8;
  text-align: center;
  line-height: 1.4;
  font-family: 'JetBrains Mono', monospace;
}

.pri-foot {
  font-size: 0.72em;
  color: #475569;
  font-style: italic;
  text-align: center;
}
</style>
