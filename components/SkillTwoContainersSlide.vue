<script setup>
import { useNav } from '@slidev/client'
import { computed } from 'vue'

const { clicks } = useNav()

const showWrapper = computed(() => clicks.value >= 1)
const showArrow   = computed(() => clicks.value >= 2)
const showService = computed(() => clicks.value >= 3)
</script>

<template>
  <div class="stc-root">

    <!-- Title -->
    <div class="stc-title">Каждый навык — два контейнера</div>

    <!-- Layout: OpenClaw | Skill (2 boxes) -->
    <div class="stc-layout">

      <!-- Left: OpenClaw -->
      <div class="stc-col">
        <div class="stc-label">AI-агент</div>
        <div class="stc-box stc-box--blue">OpenClaw</div>
      </div>

      <!-- Center: arrow OpenClaw → wrapper (MCP) -->
      <div class="stc-arrow-col">
        <div class="stc-arrow-label" :class="{ 'stc-visible': showWrapper }">MCP</div>
        <svg class="stc-arrow-svg" viewBox="0 0 80 20">
          <defs>
            <marker id="stc-arr-r" markerWidth="6" markerHeight="6" refX="5" refY="3" orient="auto">
              <polygon points="0,0.5 0,5.5 6,3" fill="#3b82f6"/>
            </marker>
          </defs>
          <line v-if="showWrapper" x1="4" y1="10" x2="70" y2="10"
            stroke="#3b82f6" stroke-width="1.8" stroke-linecap="round"
            marker-end="url(#stc-arr-r)"
            class="stc-line"/>
        </svg>
      </div>

      <!-- Right: Skill double-box -->
      <div class="stc-col">
        <div class="stc-label">Навык</div>
        <div class="stc-double">

          <!-- Outer: MCP wrapper -->
          <div class="stc-wrapper" :class="{ 'stc-visible': showWrapper }">
            <div class="stc-wrapper-label">MCP-обёртка</div>
            <div class="stc-wrapper-hint">принимает команды от агента</div>

            <!-- Inner arrow wrapper → service -->
            <div class="stc-inner-arrow" :class="{ 'stc-visible': showArrow }">
              <svg viewBox="0 0 20 40" class="stc-iarrow-svg">
                <defs>
                  <marker id="stc-arr-d" markerWidth="6" markerHeight="6" refX="5" refY="3" orient="auto">
                    <polygon points="0,0.5 0,5.5 6,3" fill="#475569"/>
                  </marker>
                </defs>
                <line x1="10" y1="2" x2="10" y2="32"
                  stroke="#475569" stroke-width="1.5"
                  stroke-dasharray="3 3"
                  marker-end="url(#stc-arr-d)"/>
              </svg>
              <span class="stc-inner-label">HTTP</span>
            </div>

            <!-- Inner: Service -->
            <div class="stc-service" :class="{ 'stc-visible': showService }">
              <div class="stc-service-label">Сервис</div>
              <div class="stc-service-hint">делает реальную работу</div>
            </div>
          </div>
        </div>
      </div>

    </div>

    <!-- Bottom hint -->
    <div class="stc-hint" :class="{ 'stc-visible': showService }">
      Агент знает только о MCP-интерфейсе — что внутри, его не касается
    </div>

  </div>
</template>

<style scoped>
.stc-root {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 28px 48px 20px;
  box-sizing: border-box;
  gap: 24px;
}

.stc-title {
  font-size: 1.1em;
  font-weight: 700;
  color: #93c5fd;
  letter-spacing: 0.02em;
  align-self: flex-start;
}

.stc-layout {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0;
  width: 100%;
}

.stc-col {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
}

.stc-label {
  font-size: 0.75em;
  color: #64748b;
  text-transform: uppercase;
  letter-spacing: 0.08em;
}

.stc-box {
  padding: 14px 28px;
  border-radius: 10px;
  font-weight: 700;
  font-size: 1.25em;
  text-align: center;
}

.stc-box--blue {
  background: rgba(37, 99, 235, 0.18);
  border: 2px solid #3b82f6;
  color: #93c5fd;
  box-shadow: 0 0 18px rgba(59, 130, 246, 0.22);
}

/* Arrow column */
.stc-arrow-col {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100px;
  gap: 4px;
}

.stc-arrow-label {
  font-size: 0.72em;
  color: #3b82f6;
  letter-spacing: 0.06em;
  opacity: 0;
  transition: opacity 0.3s;
}

.stc-arrow-label.stc-visible { opacity: 1; }

.stc-arrow-svg {
  width: 80px;
  height: 20px;
}

.stc-line {
  stroke-dasharray: 100;
  stroke-dashoffset: 100;
  animation: stc-draw 0.4s ease-out both;
}

@keyframes stc-draw {
  to { stroke-dashoffset: 0; }
}

/* Double box: wrapper + service */
.stc-double {
  display: flex;
  flex-direction: column;
}

.stc-wrapper {
  border: 2px solid #475569;
  border-radius: 10px;
  background: rgba(71, 85, 105, 0.10);
  padding: 16px 20px 12px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  opacity: 0;
  transform: scale(0.96);
  min-width: 220px;
}

.stc-wrapper.stc-visible {
  animation: stc-pop 0.35s ease-out both;
}

@keyframes stc-pop {
  from { opacity: 0; transform: scale(0.96); }
  to   { opacity: 1; transform: scale(1); }
}

.stc-wrapper-label {
  font-weight: 700;
  font-size: 0.95em;
  color: #94a3b8;
}

.stc-wrapper-hint {
  font-size: 0.70em;
  color: #64748b;
}

.stc-inner-arrow {
  display: flex;
  align-items: center;
  gap: 6px;
  opacity: 0;
  transition: opacity 0.3s;
}

.stc-inner-arrow.stc-visible { opacity: 1; }

.stc-iarrow-svg {
  width: 20px;
  height: 32px;
}

.stc-inner-label {
  font-size: 0.68em;
  color: #475569;
  letter-spacing: 0.06em;
}

.stc-service {
  border: 1.5px solid #334155;
  border-radius: 8px;
  background: rgba(51, 65, 85, 0.18);
  padding: 12px 20px 10px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  width: 100%;
  box-sizing: border-box;
  opacity: 0;
  transform: translateY(6px);
}

.stc-service.stc-visible {
  animation: stc-appear 0.35s ease-out both;
}

@keyframes stc-appear {
  from { opacity: 0; transform: translateY(6px); }
  to   { opacity: 1; transform: translateY(0); }
}

.stc-service-label {
  font-weight: 700;
  font-size: 0.92em;
  color: #7c8fa8;
}

.stc-service-hint {
  font-size: 0.68em;
  color: #475569;
}

.stc-hint {
  font-size: 0.78em;
  color: #64748b;
  text-align: center;
  opacity: 0;
  transition: opacity 0.4s;
  font-style: italic;
}

.stc-hint.stc-visible { opacity: 1; }
</style>
