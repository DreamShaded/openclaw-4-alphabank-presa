<script setup>
import { useNav } from '@slidev/client'
import { computed } from 'vue'

const { clicks } = useNav()

const showInit  = computed(() => clicks.value >= 1)
const showDisc  = computed(() => clicks.value >= 2)
const showOp    = computed(() => clicks.value >= 3)
const showShut  = computed(() => clicks.value >= 4)
</script>

<template>
  <div class="mlc-root">

    <div class="mlc-title">Жизненный цикл сессии</div>

    <div class="mlc-flow">

      <!-- Phase 1 -->
      <div class="mlc-step" :class="{ 'mlc-visible': showInit }">
        <div class="mlc-step-num">1</div>
        <div class="mlc-step-icon">🤝</div>
        <div class="mlc-step-name">Initialize</div>
        <div class="mlc-step-desc">обязательное<br/>рукопожатие</div>
      </div>

      <div class="mlc-conn" :class="{ 'mlc-visible': showDisc }"></div>

      <!-- Phase 2 -->
      <div class="mlc-step" :class="{ 'mlc-visible': showDisc }">
        <div class="mlc-step-num">2</div>
        <div class="mlc-step-icon">🔍</div>
        <div class="mlc-step-name">Discovery</div>
        <div class="mlc-step-desc">клиент узнаёт,<br/>что умеет сервер</div>
      </div>

      <div class="mlc-conn" :class="{ 'mlc-visible': showOp }"></div>

      <!-- Phase 3 -->
      <div class="mlc-step mlc-step--main" :class="{ 'mlc-visible': showOp }">
        <div class="mlc-step-num">3</div>
        <div class="mlc-step-icon">⚙️</div>
        <div class="mlc-step-name">Operation</div>
        <div class="mlc-step-desc">основная работа:<br/>requests · notifications</div>
      </div>

      <div class="mlc-conn" :class="{ 'mlc-visible': showShut }"></div>

      <!-- Phase 4 -->
      <div class="mlc-step" :class="{ 'mlc-visible': showShut }">
        <div class="mlc-step-num">4</div>
        <div class="mlc-step-icon">👋</div>
        <div class="mlc-step-name">Shutdown</div>
        <div class="mlc-step-desc">graceful close,<br/>любая сторона</div>
      </div>

    </div>

    <div class="mlc-foot" :class="{ 'mlc-visible': showShut }">
      между шагами 1–4 сессия живёт минуты, часы, дни — без переподключений
    </div>

  </div>
</template>

<style scoped>
.mlc-root {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 28px 36px 24px;
  box-sizing: border-box;
  gap: 28px;
}

.mlc-title {
  font-size: 1.1em;
  font-weight: 700;
  color: #93c5fd;
  letter-spacing: 0.02em;
  align-self: flex-start;
}

/* Horizontal flow */
.mlc-flow {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0;
  width: 100%;
}

/* Phase card */
.mlc-step {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  padding: 16px 18px 14px;
  border-radius: 12px;
  background: rgba(71, 85, 105, 0.10);
  border: 1.5px solid #475569;
  min-width: 150px;
  opacity: 0;
  transform: translateY(10px) scale(0.96);
}

.mlc-step.mlc-visible {
  animation: mlc-pop 0.38s ease-out both;
}

@keyframes mlc-pop {
  from { opacity: 0; transform: translateY(10px) scale(0.96); }
  to   { opacity: 1; transform: translateY(0) scale(1); }
}

.mlc-step--main {
  background: rgba(37, 99, 235, 0.16);
  border: 2px solid #3b82f6;
  box-shadow: 0 0 18px rgba(59, 130, 246, 0.22);
}

.mlc-step-num {
  position: absolute;
  top: -10px;
  left: -10px;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: #3b82f6;
  color: #fff;
  font-size: 0.78em;
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: center;
}

.mlc-step-icon {
  font-size: 1.6em;
  line-height: 1;
}

.mlc-step-name {
  font-weight: 700;
  font-size: 0.95em;
  color: #e2e8f0;
  letter-spacing: 0.02em;
}

.mlc-step-desc {
  font-size: 0.7em;
  color: #94a3b8;
  text-align: center;
  line-height: 1.4;
}

/* Connector */
.mlc-conn {
  flex: 1;
  max-width: 60px;
  height: 2px;
  background: linear-gradient(90deg, #475569 0%, #3b82f6 50%, #475569 100%);
  opacity: 0;
  transform-origin: left center;
  transform: scaleX(0);
  margin: 0 4px;
}

.mlc-conn.mlc-visible {
  animation: mlc-line 0.35s ease-out both;
}

@keyframes mlc-line {
  from { opacity: 0; transform: scaleX(0); }
  to   { opacity: 1; transform: scaleX(1); }
}

.mlc-foot {
  font-size: 0.74em;
  color: #475569;
  font-style: italic;
  opacity: 0;
  transition: opacity 0.4s;
  text-align: center;
}

.mlc-foot.mlc-visible { opacity: 1; }
</style>
