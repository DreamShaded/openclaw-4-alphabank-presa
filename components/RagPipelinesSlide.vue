<script setup>
import { useNav } from '@slidev/client'
import { computed } from 'vue'

const { clicks } = useNav()

const showOffline = computed(() => clicks.value >= 1)
const showOnline  = computed(() => clicks.value >= 2)
</script>

<template>
  <div class="rp-root">

    <div class="rp-title">Два пайплайна RAG</div>

    <div class="rp-pipelines">

      <!-- Offline -->
      <div class="rp-row rp-row--offline" :class="{ 'rp-visible': showOffline }">
        <div class="rp-side">
          <div class="rp-side-tag">OFFLINE</div>
          <div class="rp-side-name">Индексация</div>
          <div class="rp-side-when">делается заранее, один раз</div>
        </div>

        <div class="rp-flow">
          <div class="rp-step">
            <div class="rp-step-icon">📚</div>
            <div class="rp-step-name">заметки</div>
          </div>
          <div class="rp-arrow">→</div>
          <div class="rp-step">
            <div class="rp-step-icon">✂️</div>
            <div class="rp-step-name">режем<br/>на куски</div>
          </div>
          <div class="rp-arrow">→</div>
          <div class="rp-step">
            <div class="rp-step-icon">🔢</div>
            <div class="rp-step-name">превращаем<br/>в векторы</div>
          </div>
          <div class="rp-arrow">→</div>
          <div class="rp-step rp-step--db">
            <div class="rp-step-icon">🗄️</div>
            <div class="rp-step-name">векторная<br/>БД</div>
          </div>
        </div>
      </div>

      <!-- Online -->
      <div class="rp-row rp-row--online" :class="{ 'rp-visible': showOnline }">
        <div class="rp-side">
          <div class="rp-side-tag rp-side-tag--online">ONLINE</div>
          <div class="rp-side-name">Запрос</div>
          <div class="rp-side-when">на каждый вопрос пользователя</div>
        </div>

        <div class="rp-flow">
          <div class="rp-step">
            <div class="rp-step-icon">❓</div>
            <div class="rp-step-name">вопрос</div>
          </div>
          <div class="rp-arrow rp-arrow--blue">→</div>
          <div class="rp-step">
            <div class="rp-step-icon">🔢</div>
            <div class="rp-step-name">в вектор</div>
          </div>
          <div class="rp-arrow rp-arrow--blue">→</div>
          <div class="rp-step rp-step--db">
            <div class="rp-step-icon">🔍</div>
            <div class="rp-step-name">поиск<br/>в БД</div>
          </div>
          <div class="rp-arrow rp-arrow--blue">→</div>
          <div class="rp-step rp-step--llm">
            <div class="rp-step-icon">🤖</div>
            <div class="rp-step-name">LLM<br/>+ контекст</div>
          </div>
          <div class="rp-arrow rp-arrow--blue">→</div>
          <div class="rp-step rp-step--answer">
            <div class="rp-step-icon">💬</div>
            <div class="rp-step-name">ответ</div>
          </div>
        </div>
      </div>

    </div>

    <div class="rp-foot" :class="{ 'rp-visible': showOnline }">
      БД общая для обоих контуров — записали один раз, читаем много раз
    </div>

  </div>
</template>

<style scoped>
.rp-root {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 22px 28px 14px;
  box-sizing: border-box;
  gap: 16px;
}

.rp-title {
  font-size: 1.1em;
  font-weight: 700;
  color: #93c5fd;
  letter-spacing: 0.02em;
  align-self: flex-start;
}

.rp-pipelines {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  gap: 22px;
  width: 100%;
}

/* Each pipeline row */
.rp-row {
  display: flex;
  align-items: center;
  gap: 18px;
  padding: 14px 16px;
  border-radius: 12px;
  opacity: 0;
  transform: translateY(8px);
}

.rp-row.rp-visible {
  animation: rp-appear 0.4s ease-out both;
}

@keyframes rp-appear {
  from { opacity: 0; transform: translateY(8px); }
  to   { opacity: 1; transform: translateY(0); }
}

.rp-row--offline {
  background: rgba(168, 85, 247, 0.08);
  border: 1.5px solid rgba(168, 85, 247, 0.4);
}

.rp-row--online {
  background: rgba(37, 99, 235, 0.12);
  border: 2px solid #3b82f6;
  box-shadow: 0 0 18px rgba(59, 130, 246, 0.20);
}

/* Left side label */
.rp-side {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  min-width: 130px;
  gap: 2px;
}

.rp-side-tag {
  display: inline-block;
  background: rgba(168, 85, 247, 0.25);
  border: 1px solid #a855f7;
  color: #c4b5fd;
  padding: 2px 10px;
  border-radius: 999px;
  font-size: 0.62em;
  font-weight: 700;
  letter-spacing: 0.1em;
}

.rp-side-tag--online {
  background: rgba(59, 130, 246, 0.25);
  border-color: #3b82f6;
  color: #93c5fd;
}

.rp-side-name {
  font-size: 0.9em;
  font-weight: 700;
  color: #e2e8f0;
  margin-top: 4px;
}

.rp-side-when {
  font-size: 0.65em;
  color: #64748b;
  font-style: italic;
}

/* Flow */
.rp-flow {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
}

.rp-step {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  padding: 8px 10px;
  border-radius: 8px;
  background: rgba(15, 23, 42, 0.5);
  border: 1px solid #334155;
  min-width: 68px;
}

.rp-step--db {
  border-color: #3b82f6;
  background: rgba(37, 99, 235, 0.15);
}

.rp-step--llm {
  border-color: #10b981;
  background: rgba(16, 185, 129, 0.15);
}

.rp-step--answer {
  border-color: #10b981;
  background: rgba(16, 185, 129, 0.15);
}

.rp-step-icon {
  font-size: 1.2em;
  line-height: 1;
}

.rp-step-name {
  font-size: 0.65em;
  color: #cbd5e1;
  text-align: center;
  line-height: 1.3;
}

.rp-arrow {
  font-size: 1.1em;
  color: #a855f7;
  font-weight: 700;
}

.rp-arrow--blue {
  color: #60a5fa;
}

.rp-foot {
  font-size: 0.72em;
  color: #475569;
  font-style: italic;
  text-align: center;
  opacity: 0;
  transition: opacity 0.4s;
}

.rp-foot.rp-visible { opacity: 1; }
</style>
