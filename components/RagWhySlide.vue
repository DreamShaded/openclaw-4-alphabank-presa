<script setup>
import { useNav } from '@slidev/client'
import { computed } from 'vue'

const { clicks } = useNav()

const showProblem  = computed(() => clicks.value >= 1)
const showSolution = computed(() => clicks.value >= 2)
</script>

<template>
  <div class="rw-root">

    <div class="rw-title">RAG — почему он нужен</div>

    <div class="rw-grid">

      <!-- Problem: LLM alone -->
      <div class="rw-col rw-col--bad" :class="{ 'rw-visible': showProblem }">
        <div class="rw-col-label">Без RAG</div>

        <div class="rw-flow">
          <div class="rw-q">«где у меня заметки про MCP?»</div>
          <div class="rw-arrow">↓</div>
          <div class="rw-llm">🤖 LLM</div>
          <div class="rw-arrow">↓</div>
          <div class="rw-bad-answer">
            «извините, я не знаю<br/>про ваши заметки»
          </div>
        </div>

        <div class="rw-col-hint">модель видит только то,<br/>на чём её обучили</div>
      </div>

      <!-- Solution: LLM + RAG -->
      <div class="rw-col rw-col--good" :class="{ 'rw-visible': showSolution }">
        <div class="rw-col-label rw-col-label--accent">С RAG</div>

        <div class="rw-flow">
          <div class="rw-q">«где у меня заметки про MCP?»</div>
          <div class="rw-arrow">↓</div>
          <div class="rw-retrieval">
            <span class="rw-retrieval-icon">🔎</span>
            <span>находим релевантные куски<br/>в базе знаний</span>
          </div>
          <div class="rw-arrow rw-arrow--blue">↓ + контекст</div>
          <div class="rw-llm rw-llm--accent">🤖 LLM</div>
          <div class="rw-arrow rw-arrow--blue">↓</div>
          <div class="rw-good-answer">
            «вот три заметки: <br/>2026-05-12, 2026-04-08...»
          </div>
        </div>

        <div class="rw-col-hint rw-col-hint--accent">модель отвечает<br/>на основе ваших данных</div>
      </div>

    </div>

    <div class="rw-foot">
      RAG = <b>R</b>etrieval-<b>A</b>ugmented <b>G</b>eneration · ответ из весов <b>+</b> ответ из ваших документов
    </div>

  </div>
</template>

<style scoped>
.rw-root {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 22px 32px 14px;
  box-sizing: border-box;
  gap: 14px;
}

.rw-title {
  font-size: 1.1em;
  font-weight: 700;
  color: #93c5fd;
  letter-spacing: 0.02em;
  align-self: flex-start;
}

.rw-grid {
  flex: 1;
  display: flex;
  align-items: stretch;
  justify-content: center;
  gap: 28px;
  width: 100%;
}

.rw-col {
  flex: 1;
  max-width: 360px;
  border-radius: 12px;
  padding: 14px 18px 12px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  opacity: 0;
  transform: translateY(10px);
}

.rw-col.rw-visible {
  animation: rw-appear 0.4s ease-out both;
}

@keyframes rw-appear {
  from { opacity: 0; transform: translateY(10px); }
  to   { opacity: 1; transform: translateY(0); }
}

.rw-col--bad {
  background: rgba(71, 85, 105, 0.10);
  border: 1.5px solid #475569;
}

.rw-col--good {
  background: rgba(37, 99, 235, 0.14);
  border: 2px solid #3b82f6;
  box-shadow: 0 0 18px rgba(59, 130, 246, 0.22);
}

.rw-col-label {
  font-size: 1em;
  font-weight: 700;
  color: #94a3b8;
  letter-spacing: 0.04em;
}

.rw-col-label--accent {
  color: #93c5fd;
}

/* Flow inside column */
.rw-flow {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  width: 100%;
}

.rw-q {
  background: rgba(15, 23, 42, 0.5);
  border: 1px solid #334155;
  border-radius: 6px;
  padding: 6px 12px;
  font-size: 0.75em;
  color: #cbd5e1;
  font-style: italic;
  text-align: center;
}

.rw-arrow {
  font-size: 0.85em;
  color: #475569;
  line-height: 1;
}

.rw-arrow--blue { color: #60a5fa; font-size: 0.7em; letter-spacing: 0.04em; }

.rw-llm {
  background: rgba(71, 85, 105, 0.20);
  border: 1px solid #475569;
  border-radius: 6px;
  padding: 6px 16px;
  font-size: 0.85em;
  font-weight: 600;
  color: #94a3b8;
}

.rw-llm--accent {
  background: rgba(37, 99, 235, 0.22);
  border-color: #3b82f6;
  color: #93c5fd;
}

.rw-retrieval {
  display: flex;
  align-items: center;
  gap: 8px;
  background: rgba(168, 85, 247, 0.12);
  border: 1px solid #a855f7;
  border-radius: 6px;
  padding: 6px 12px;
  font-size: 0.72em;
  color: #c4b5fd;
  text-align: left;
}

.rw-retrieval-icon { font-size: 1.1em; }

.rw-bad-answer {
  background: rgba(239, 68, 68, 0.12);
  border: 1px dashed #ef4444;
  border-radius: 6px;
  padding: 6px 12px;
  font-size: 0.72em;
  color: #fca5a5;
  font-style: italic;
  text-align: center;
}

.rw-good-answer {
  background: rgba(16, 185, 129, 0.12);
  border: 1px solid #10b981;
  border-radius: 6px;
  padding: 6px 12px;
  font-size: 0.72em;
  color: #6ee7b7;
  text-align: center;
}

.rw-col-hint {
  font-size: 0.7em;
  color: #64748b;
  text-align: center;
  font-style: italic;
  margin-top: 4px;
}

.rw-col-hint--accent { color: #60a5fa; }

.rw-foot {
  font-size: 0.74em;
  color: #475569;
  text-align: center;
  font-style: italic;
}

.rw-foot b {
  color: #93c5fd;
  font-weight: 700;
}
</style>
