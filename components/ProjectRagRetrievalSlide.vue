<script setup>
import { useNav } from '@slidev/client'
import { computed } from 'vue'

const { clicks } = useNav()

const showHybrid = computed(() => clicks.value >= 1)
const showRerank = computed(() => clicks.value >= 2)
const showGraph  = computed(() => clicks.value >= 3)
const showLlm    = computed(() => clicks.value >= 4)
const showAns    = computed(() => clicks.value >= 5)
</script>

<template>
  <div class="prr-root">

    <div class="prr-title">Из вопроса в ответ</div>

    <div class="prr-flow">

      <!-- Question -->
      <div class="prr-q">
        <span class="prr-q-icon">❓</span>
        <span class="prr-q-text">«что мы решили по интеграции?»</span>
      </div>

      <div class="prr-arrow prr-arrow--on">↓</div>

      <!-- Hybrid search -->
      <div class="prr-step prr-step--blue" :class="{ 'prr-on': showHybrid }">
        <div class="prr-step-row">
          <span class="prr-step-icon">🔍</span>
          <span class="prr-step-name">Hybrid search в Qdrant</span>
        </div>
        <div class="prr-step-meta">dense + sparse · слияние RRF · top-K кусков</div>
      </div>

      <div class="prr-arrow" :class="{ 'prr-arrow--on': showRerank }">↓</div>

      <!-- Rerank -->
      <div class="prr-step prr-step--blue" :class="{ 'prr-on': showRerank }">
        <div class="prr-step-row">
          <span class="prr-step-icon">🎯</span>
          <span class="prr-step-name">Rerank</span>
        </div>
        <div class="prr-step-meta">BGE-reranker · точная пересортировка</div>
      </div>

      <div class="prr-arrow" :class="{ 'prr-arrow--on': showGraph }">↓</div>

      <!-- Graph expansion -->
      <div class="prr-step prr-step--purple" :class="{ 'prr-on': showGraph }">
        <div class="prr-step-row">
          <span class="prr-step-icon">🕸️</span>
          <span class="prr-step-name">Graph expansion из SQLite</span>
        </div>
        <div class="prr-step-meta">тянем связанные сущности и их чанки</div>
      </div>

      <div class="prr-arrow" :class="{ 'prr-arrow--on': showLlm }">↓ <span class="prr-loop">↺ до 3 hops (IRCoT)</span></div>

      <!-- LLM synth -->
      <div class="prr-step prr-step--green" :class="{ 'prr-on': showLlm }">
        <div class="prr-step-row">
          <span class="prr-step-icon">🤖</span>
          <span class="prr-step-name">LLM (Kimi/Moonshot)</span>
        </div>
        <div class="prr-step-meta">собирает контекст · синтезирует ответ</div>
      </div>

      <div class="prr-arrow" :class="{ 'prr-arrow--on': showAns }">↓</div>

      <!-- Answer -->
      <div class="prr-answer" :class="{ 'prr-on': showAns }">
        <span class="prr-q-icon">💬</span>
        <span>ответ <span class="prr-answer-cite">+ цитаты-источники</span></span>
      </div>

    </div>

  </div>
</template>

<style scoped>
.prr-root {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 22px 28px 14px;
  box-sizing: border-box;
  gap: 12px;
}

.prr-title {
  font-size: 1.1em;
  font-weight: 700;
  color: #93c5fd;
  letter-spacing: 0.02em;
  align-self: flex-start;
}

.prr-flow {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 4px;
  width: 100%;
  max-width: 560px;
}

/* Question / answer */
.prr-q,
.prr-answer {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 8px 18px;
  border-radius: 8px;
  background: rgba(15, 23, 42, 0.5);
  border: 1px solid #334155;
  font-size: 0.85em;
  color: #cbd5e1;
  font-style: italic;
}

.prr-answer {
  background: rgba(16, 185, 129, 0.12);
  border: 1.5px solid #10b981;
  color: #6ee7b7;
  font-style: normal;
  font-weight: 600;
  opacity: 0;
  transform: scale(0.96);
  transition: opacity 0.35s, transform 0.35s;
}

.prr-answer.prr-on {
  opacity: 1;
  transform: scale(1);
}

.prr-answer-cite {
  color: #94a3b8;
  font-weight: 400;
  font-size: 0.9em;
  font-style: italic;
}

.prr-q-icon { font-size: 1em; }

/* Step boxes */
.prr-step {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
  padding: 8px 16px;
  border-radius: 10px;
  opacity: 0.3;
  transform: translateY(4px);
  transition: opacity 0.35s, transform 0.35s;
}

.prr-step.prr-on {
  opacity: 1;
  transform: translateY(0);
}

.prr-step--blue {
  background: rgba(37, 99, 235, 0.14);
  border: 1.5px solid #3b82f6;
}

.prr-step--purple {
  background: rgba(168, 85, 247, 0.12);
  border: 1.5px solid #a855f7;
}

.prr-step--green {
  background: rgba(16, 185, 129, 0.12);
  border: 1.5px solid #10b981;
}

.prr-step-row {
  display: flex;
  align-items: center;
  gap: 8px;
}

.prr-step-icon { font-size: 1.05em; }

.prr-step-name {
  font-weight: 700;
  font-size: 0.86em;
  color: #e2e8f0;
}

.prr-step-meta {
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.62em;
  color: #94a3b8;
  letter-spacing: 0.02em;
}

/* Arrows */
.prr-arrow {
  font-size: 0.95em;
  color: #334155;
  font-weight: 700;
  line-height: 1;
  display: flex;
  align-items: center;
  gap: 8px;
  transition: color 0.35s;
}

.prr-arrow.prr-arrow--on { color: #3b82f6; }

.prr-loop {
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.64em;
  color: #a855f7;
  font-weight: 500;
  letter-spacing: 0.02em;
}
</style>
