<script setup>
import { useNav } from '@slidev/client'
import { computed } from 'vue'

const { clicks } = useNav()

const showMcp   = computed(() => clicks.value >= 1)
const showSkill1 = computed(() => clicks.value >= 2)
const showSkill2 = computed(() => clicks.value >= 3)
const showSkill3 = computed(() => clicks.value >= 4)
</script>

<template>
  <div class="csa-root">

    <!-- OpenClaw -->
    <div class="csa-agent">
      <div class="csa-label-top">AI-агент</div>
      <div class="csa-box csa-box--blue">OpenClaw</div>
    </div>

    <!-- MCP protocol label -->
    <div class="csa-proto" :class="{ 'csa-visible': showMcp }">
      MCP (Streamable-HTTP)
    </div>

    <!-- Skills row -->
    <div class="csa-skills">
      <div class="csa-skill" :class="{ 'csa-visible': showSkill1 }">
        <div class="csa-docker-badge">🐳 Docker</div>
        <div class="csa-box csa-box--gray">Навык A</div>
      </div>
      <div class="csa-skill" :class="{ 'csa-visible': showSkill2 }">
        <div class="csa-docker-badge">🐳 Docker</div>
        <div class="csa-box csa-box--gray">Навык B</div>
      </div>
      <div class="csa-skill" :class="{ 'csa-visible': showSkill3 }">
        <div class="csa-docker-badge">🐳 Docker</div>
        <div class="csa-box csa-box--gray">Навык C</div>
      </div>
    </div>

    <!-- SVG arrows -->
    <svg class="csa-svg" viewBox="0 0 980 552" xmlns="http://www.w3.org/2000/svg">
      <defs>
        <marker id="csa-arr" markerWidth="7" markerHeight="7" refX="6" refY="3.5" orient="auto">
          <polygon points="0,0.5 0,6.5 7,3.5" fill="#3b82f6"/>
        </marker>
      </defs>
      <!-- Down to skill A: starts from left third of wide box -->
      <path v-if="showSkill1" class="csa-path"
        d="M300,175 C300,290 210,320 210,370"
        pathLength="1" marker-end="url(#csa-arr)"/>
      <!-- Down to skill B: starts from center of box -->
      <path v-if="showSkill2" class="csa-path"
        d="M490,175 L490,370"
        pathLength="1" marker-end="url(#csa-arr)"/>
      <!-- Down to skill C: starts from right third of wide box -->
      <path v-if="showSkill3" class="csa-path"
        d="M680,175 C680,290 770,320 770,370"
        pathLength="1" marker-end="url(#csa-arr)"/>
    </svg>

  </div>
</template>

<style scoped>
.csa-root {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 32px 48px 24px;
  box-sizing: border-box;
  gap: 0;
}

.csa-agent {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  margin-top: 12px;
}

.csa-label-top {
  font-size: 0.78em;
  color: #64748b;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

.csa-box {
  padding: 14px 32px;
  border-radius: 10px;
  font-weight: 700;
  font-size: 1.35em;
  text-align: center;
  user-select: none;
  min-width: 180px;
}

.csa-box--blue {
  background: rgba(37, 99, 235, 0.18);
  border: 2px solid #3b82f6;
  color: #93c5fd;
  box-shadow: 0 0 18px rgba(59, 130, 246, 0.22);
  width: 540px;
}

.csa-box--gray {
  background: rgba(71, 85, 105, 0.12);
  border: 1.5px solid #475569;
  color: #94a3b8;
  font-size: 0.95em;
}

.csa-proto {
  margin-top: 16px;
  font-size: 0.82em;
  color: #3b82f6;
  letter-spacing: 0.06em;
  opacity: 0;
  transition: opacity 0.3s ease;
  padding: 4px 16px;
  border: 1px dashed rgba(59,130,246,0.4);
  border-radius: 20px;
}

.csa-proto.csa-visible {
  opacity: 1;
}

.csa-skills {
  display: flex;
  gap: 48px;
  margin-top: 120px;
  align-items: flex-end;
}

.csa-skill {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  opacity: 0;
  transform: translateY(12px);
}

.csa-skill.csa-visible {
  animation: csa-appear 0.35s ease-out both;
}

@keyframes csa-appear {
  from { opacity: 0; transform: translateY(12px); }
  to   { opacity: 1; transform: translateY(0); }
}

.csa-docker-badge {
  font-size: 0.72em;
  color: #64748b;
  letter-spacing: 0.05em;
}

.csa-svg {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

.csa-path {
  fill: none;
  stroke: #3b82f6;
  stroke-width: 1.8;
  stroke-linecap: round;
  stroke-dasharray: 1;
  stroke-dashoffset: 1;
  animation: csa-draw 0.5s ease-out both;
}

@keyframes csa-draw {
  to { stroke-dashoffset: 0; }
}
</style>
