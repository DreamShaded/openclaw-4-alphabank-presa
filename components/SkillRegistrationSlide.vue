<script setup>
import { useNav } from '@slidev/client'
import { computed } from 'vue'

const { clicks } = useNav()

const showCompose = computed(() => clicks.value >= 1)
const showArrow1  = computed(() => clicks.value >= 2)
const showConfig  = computed(() => clicks.value >= 3)
const showArrow2  = computed(() => clicks.value >= 4)
const showReady   = computed(() => clicks.value >= 5)
</script>

<template>
  <div class="sreg-root">

    <div class="sreg-title">Как навык попадает в агента</div>

    <div class="sreg-flow">

      <!-- Step 1: docker-compose.yml -->
      <div class="sreg-step" :class="{ 'sreg-visible': showCompose }">
        <div class="sreg-step-icon">📄</div>
        <div class="sreg-box sreg-box--gray">
          <div class="sreg-box-title">docker-compose.yml</div>
          <pre class="sreg-code">skill-mcp:
  image: my-skill-mcp
  ports: ["3100:3100"]</pre>
        </div>
        <div class="sreg-step-label">описываем навык в Compose</div>
      </div>

      <!-- Arrow 1 -->
      <div class="sreg-arrow" :class="{ 'sreg-visible': showArrow1 }">
        <svg viewBox="0 0 40 24" width="40" height="24">
          <defs>
            <marker id="sreg-arr" markerWidth="6" markerHeight="6" refX="5" refY="3" orient="auto">
              <polygon points="0,0.5 0,5.5 6,3" fill="#3b82f6"/>
            </marker>
          </defs>
          <line x1="4" y1="12" x2="32" y2="12" stroke="#3b82f6" stroke-width="1.8"
            marker-end="url(#sreg-arr)" class="sreg-line"/>
        </svg>
      </div>

      <!-- Step 2: openclaw config -->
      <div class="sreg-step" :class="{ 'sreg-visible': showConfig }">
        <div class="sreg-step-icon">⚙️</div>
        <div class="sreg-box sreg-box--gray">
          <div class="sreg-box-title">config/openclaw.json</div>
          <pre class="sreg-code">"mcpServers": {
  "my-skill": {
    "url": "http://skill-mcp:3100/mcp"
  }
}</pre>
        </div>
        <div class="sreg-step-label">регистрируем MCP-адрес в конфиге</div>
      </div>

      <!-- Arrow 2 -->
      <div class="sreg-arrow" :class="{ 'sreg-visible': showArrow2 }">
        <svg viewBox="0 0 40 24" width="40" height="24">
          <line x1="4" y1="12" x2="32" y2="12" stroke="#3b82f6" stroke-width="1.8"
            marker-end="url(#sreg-arr)" class="sreg-line"/>
        </svg>
      </div>

      <!-- Step 3: OpenClaw knows -->
      <div class="sreg-step" :class="{ 'sreg-visible': showReady }">
        <div class="sreg-step-icon">✅</div>
        <div class="sreg-box sreg-box--blue">
          <div class="sreg-box-title">OpenClaw</div>
          <div class="sreg-box-sub">навык доступен на старте сессии</div>
        </div>
        <div class="sreg-step-label">агент автоматически подключает навык</div>
      </div>

    </div>

  </div>
</template>

<style scoped>
.sreg-root {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 28px 32px 20px;
  box-sizing: border-box;
  gap: 20px;
}

.sreg-title {
  font-size: 1.1em;
  font-weight: 700;
  color: #93c5fd;
  letter-spacing: 0.02em;
  align-self: flex-start;
}

.sreg-flow {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  width: 100%;
}

.sreg-step {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  opacity: 0;
  transform: translateY(10px);
}

.sreg-step.sreg-visible {
  animation: sreg-appear 0.38s ease-out both;
}

@keyframes sreg-appear {
  from { opacity: 0; transform: translateY(10px); }
  to   { opacity: 1; transform: translateY(0); }
}

.sreg-step-icon {
  font-size: 1.4em;
}

.sreg-box {
  border-radius: 10px;
  padding: 12px 16px;
  min-width: 200px;
  text-align: left;
}

.sreg-box--gray {
  background: rgba(71, 85, 105, 0.12);
  border: 1.5px solid #475569;
}

.sreg-box--blue {
  background: rgba(37, 99, 235, 0.18);
  border: 2px solid #3b82f6;
  box-shadow: 0 0 18px rgba(59, 130, 246, 0.22);
  text-align: center;
}

.sreg-box-title {
  font-weight: 700;
  font-size: 0.88em;
  color: #94a3b8;
  margin-bottom: 8px;
}

.sreg-box-sub {
  font-size: 0.80em;
  color: #93c5fd;
  margin-top: 6px;
}

.sreg-code {
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.70em;
  color: #64748b;
  background: rgba(15, 23, 42, 0.5);
  border-radius: 6px;
  padding: 8px 10px;
  margin: 0;
  line-height: 1.5;
  white-space: pre;
}

.sreg-step-label {
  font-size: 0.68em;
  color: #475569;
  text-align: center;
  max-width: 180px;
}

/* Arrows */
.sreg-arrow {
  opacity: 0;
  transition: opacity 0.3s;
  padding-bottom: 40px;
}

.sreg-arrow.sreg-visible { opacity: 1; }

.sreg-line {
  stroke-dasharray: 100;
  stroke-dashoffset: 100;
  animation: sreg-draw 0.4s ease-out both;
}

@keyframes sreg-draw {
  to { stroke-dashoffset: 0; }
}
</style>
