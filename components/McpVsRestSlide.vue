<script setup>
import { useNav } from '@slidev/client'
import { computed } from 'vue'

const { clicks } = useNav()

const showRest     = computed(() => clicks.value >= 1)
const showMcp      = computed(() => clicks.value >= 2)
const showFeatures = computed(() => clicks.value >= 3)
</script>

<template>
  <div class="mvr-root">

    <div class="mvr-title">Почему MCP — это не REST</div>

    <div class="mvr-grid">

      <!-- REST column -->
      <div class="mvr-col mvr-col--rest" :class="{ 'mvr-visible': showRest }">
        <div class="mvr-col-label">REST</div>
        <div class="mvr-col-tagline">stateless · одноразовый</div>

        <div class="mvr-flow">
          <div class="mvr-node mvr-node--gray">клиент</div>
          <div class="mvr-flow-arrows">
            <div class="mvr-arrow">→ запрос →</div>
            <div class="mvr-arrow">← ответ ←</div>
            <div class="mvr-close">✗ соединение закрыто</div>
          </div>
          <div class="mvr-node mvr-node--gray">сервер</div>
        </div>
      </div>

      <!-- Divider -->
      <div class="mvr-divider">vs</div>

      <!-- MCP column -->
      <div class="mvr-col mvr-col--mcp" :class="{ 'mvr-visible': showMcp }">
        <div class="mvr-col-label mvr-col-label--accent">MCP</div>
        <div class="mvr-col-tagline mvr-col-tagline--accent">stateful · долгоживущая сессия</div>

        <div class="mvr-flow">
          <div class="mvr-node mvr-node--blue">клиент</div>
          <div class="mvr-flow-arrows">
            <div class="mvr-arrow mvr-arrow--blue">⇄ запрос / ответ</div>
            <div class="mvr-arrow mvr-arrow--blue">← server push (notifications)</div>
            <div class="mvr-arrow mvr-arrow--blue">⇄ ... много раз ...</div>
          </div>
          <div class="mvr-node mvr-node--blue">сервер</div>
        </div>
      </div>

    </div>

    <!-- Feature chips -->
    <div class="mvr-features" :class="{ 'mvr-visible': showFeatures }">
      <div class="mvr-chip">📡 двусторонняя инициатива</div>
      <div class="mvr-chip">💾 stateful-сессия</div>
      <div class="mvr-chip">🧩 Tool · Resource · Prompt</div>
      <div class="mvr-chip">🤝 согласование возможностей</div>
    </div>

    <div class="mvr-foot">транспорт: HTTP + SSE · формат: JSON-RPC 2.0</div>

  </div>
</template>

<style scoped>
.mvr-root {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 24px 36px 16px;
  box-sizing: border-box;
  gap: 16px;
}

.mvr-title {
  font-size: 1.1em;
  font-weight: 700;
  color: #93c5fd;
  letter-spacing: 0.02em;
  align-self: flex-start;
}

/* Two columns */
.mvr-grid {
  flex: 1;
  display: flex;
  align-items: stretch;
  justify-content: center;
  gap: 24px;
  width: 100%;
}

.mvr-col {
  flex: 1;
  border-radius: 12px;
  padding: 16px 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  opacity: 0;
  transform: translateY(8px);
  max-width: 380px;
}

.mvr-col.mvr-visible {
  animation: mvr-appear 0.35s ease-out both;
}

@keyframes mvr-appear {
  from { opacity: 0; transform: translateY(8px); }
  to   { opacity: 1; transform: translateY(0); }
}

.mvr-col--rest {
  background: rgba(71, 85, 105, 0.10);
  border: 1.5px solid #475569;
}

.mvr-col--mcp {
  background: rgba(37, 99, 235, 0.14);
  border: 2px solid #3b82f6;
  box-shadow: 0 0 18px rgba(59, 130, 246, 0.22);
}

.mvr-col-label {
  font-size: 1.15em;
  font-weight: 700;
  color: #94a3b8;
  letter-spacing: 0.04em;
}

.mvr-col-label--accent {
  color: #93c5fd;
}

.mvr-col-tagline {
  font-size: 0.72em;
  color: #64748b;
  letter-spacing: 0.04em;
  margin-bottom: 6px;
}

.mvr-col-tagline--accent {
  color: #60a5fa;
}

/* Flow inside each column */
.mvr-flow {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  width: 100%;
}

.mvr-node {
  padding: 6px 16px;
  border-radius: 6px;
  font-size: 0.78em;
  font-weight: 600;
  letter-spacing: 0.04em;
}

.mvr-node--gray {
  background: rgba(71, 85, 105, 0.20);
  border: 1px solid #475569;
  color: #94a3b8;
}

.mvr-node--blue {
  background: rgba(37, 99, 235, 0.22);
  border: 1px solid #3b82f6;
  color: #93c5fd;
}

.mvr-flow-arrows {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
  padding: 4px 0;
  font-family: 'JetBrains Mono', monospace;
}

.mvr-arrow {
  font-size: 0.7em;
  color: #64748b;
  letter-spacing: 0.02em;
}

.mvr-arrow--blue {
  color: #60a5fa;
}

.mvr-close {
  font-size: 0.65em;
  color: #ef4444;
  margin-top: 4px;
  letter-spacing: 0.04em;
}

/* Divider */
.mvr-divider {
  display: flex;
  align-items: center;
  font-size: 0.9em;
  color: #475569;
  font-weight: 700;
  letter-spacing: 0.1em;
}

/* Features row */
.mvr-features {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  justify-content: center;
  opacity: 0;
  transition: opacity 0.35s;
}

.mvr-features.mvr-visible { opacity: 1; }

.mvr-chip {
  background: rgba(37, 99, 235, 0.10);
  border: 1px solid rgba(59, 130, 246, 0.35);
  color: #93c5fd;
  border-radius: 999px;
  padding: 5px 14px;
  font-size: 0.74em;
  font-weight: 500;
}

.mvr-foot {
  font-size: 0.68em;
  color: #475569;
  letter-spacing: 0.06em;
  font-style: italic;
}
</style>
