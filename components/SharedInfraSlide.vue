<script setup>
import { useNav } from '@slidev/client'
import { computed } from 'vue'

const { clicks } = useNav()

const showSkills = computed(() => clicks.value >= 1)
const showInfra  = computed(() => clicks.value >= 2)
const showLines  = computed(() => clicks.value >= 3)
</script>

<template>
  <div class="si-root">

    <div class="si-title">Общая инфраструктура в Docker-сети</div>

    <div class="si-canvas">

      <!-- Skills row (top) -->
      <div class="si-skills" :class="{ 'si-visible': showSkills }">
        <div class="si-skill-box">Навык A</div>
        <div class="si-skill-box">Навык B</div>
        <div class="si-skill-box">Навык C</div>
      </div>

      <!-- Infra cluster (bottom) -->
      <div class="si-infra" :class="{ 'si-visible': showInfra }">
        <div class="si-infra-label">Общие сервисы</div>
        <div class="si-infra-boxes">
          <div class="si-infra-box">
            <div class="si-infra-icon">🧠</div>
            <div>AI-модель</div>
          </div>
          <div class="si-infra-box">
            <div class="si-infra-icon">🗄️</div>
            <div>База знаний</div>
          </div>
          <div class="si-infra-box">
            <div class="si-infra-icon">📡</div>
            <div>Google API</div>
          </div>
        </div>
        <div class="si-infra-hint">один экземпляр — для всех навыков</div>
      </div>

      <!-- SVG connecting lines -->
      <svg v-if="showLines" class="si-svg" viewBox="0 0 980 552" xmlns="http://www.w3.org/2000/svg">
        <defs>
          <marker id="si-arr" markerWidth="6" markerHeight="6" refX="5" refY="3" orient="auto">
            <polygon points="0,0.5 0,5.5 6,3" fill="#475569"/>
          </marker>
        </defs>
        <!-- Skill A → Infra: starts just under skill box, ends at cluster top -->
        <path class="si-path"
          d="M245,140 C245,220 350,260 410,300"
          pathLength="1" marker-end="url(#si-arr)"/>
        <!-- Skill B → Infra: straight down from skill to cluster top -->
        <path class="si-path"
          d="M490,140 L490,300"
          pathLength="1" marker-end="url(#si-arr)"/>
        <!-- Skill C → Infra: starts just under skill box, ends at cluster top -->
        <path class="si-path"
          d="M735,140 C735,220 630,260 570,300"
          pathLength="1" marker-end="url(#si-arr)"/>
      </svg>

    </div>

  </div>
</template>

<style scoped>
.si-root {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 28px 48px 20px;
  box-sizing: border-box;
  gap: 16px;
}

.si-title {
  font-size: 1.1em;
  font-weight: 700;
  color: #93c5fd;
  letter-spacing: 0.02em;
  align-self: flex-start;
}

.si-canvas {
  position: relative;
  flex: 1;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
  padding-bottom: 16px;
}

/* Skills row */
.si-skills {
  display: flex;
  gap: 48px;
  opacity: 0;
  transform: translateY(-10px);
}

.si-skills.si-visible {
  animation: si-down 0.38s ease-out both;
}

@keyframes si-down {
  from { opacity: 0; transform: translateY(-10px); }
  to   { opacity: 1; transform: translateY(0); }
}

.si-skill-box {
  background: rgba(71, 85, 105, 0.12);
  border: 1.5px solid #475569;
  color: #94a3b8;
  border-radius: 10px;
  padding: 12px 32px;
  font-weight: 700;
  font-size: 0.95em;
  text-align: center;
  min-width: 130px;
}

/* Infra cluster */
.si-infra {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  border: 2px dashed rgba(59, 130, 246, 0.35);
  border-radius: 16px;
  padding: 16px 24px 14px;
  opacity: 0;
  transform: translateY(10px);
}

.si-infra.si-visible {
  animation: si-up 0.38s ease-out both;
}

@keyframes si-up {
  from { opacity: 0; transform: translateY(10px); }
  to   { opacity: 1; transform: translateY(0); }
}

.si-infra-label {
  font-size: 0.72em;
  color: #3b82f6;
  text-transform: uppercase;
  letter-spacing: 0.08em;
}

.si-infra-boxes {
  display: flex;
  gap: 20px;
}

.si-infra-box {
  background: rgba(37, 99, 235, 0.12);
  border: 1.5px solid rgba(59, 130, 246, 0.4);
  color: #93c5fd;
  border-radius: 10px;
  padding: 10px 20px;
  font-size: 0.85em;
  font-weight: 600;
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  min-width: 110px;
}

.si-infra-icon {
  font-size: 1.2em;
}

.si-infra-hint {
  font-size: 0.68em;
  color: #475569;
  font-style: italic;
}

/* SVG overlay */
.si-svg {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

.si-path {
  fill: none;
  stroke: #475569;
  stroke-width: 1.5;
  stroke-dasharray: 1;
  stroke-dashoffset: 1;
  animation: si-draw 0.5s ease-out both;
}

@keyframes si-draw {
  to { stroke-dashoffset: 0; }
}
</style>
