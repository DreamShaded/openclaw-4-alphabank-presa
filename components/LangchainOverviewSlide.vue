<script setup>
import { useNav } from '@slidev/client'
import { computed } from 'vue'

const { clicks } = useNav()

const showBlocks   = computed(() => clicks.value >= 1)
const showRunnable = computed(() => clicks.value >= 2)
</script>

<template>
  <div class="lo-root">

    <div class="lo-title">LangChain — конструктор LLM-приложений</div>
    <div class="lo-sub">готовые кирпичи для типичных паттернов · Python / JS</div>

    <!-- 4 building blocks -->
    <div class="lo-blocks" :class="{ 'lo-on': showBlocks }">
      <div class="lo-block lo-block--agent">
        <div class="lo-block-icon">🤖</div>
        <div class="lo-block-name">Agent</div>
        <div class="lo-block-desc">LLM сам решает,<br/>какой tool вызвать</div>
      </div>
      <div class="lo-block lo-block--tool">
        <div class="lo-block-icon">🔧</div>
        <div class="lo-block-name">Tool</div>
        <div class="lo-block-desc">функция<br/>с описанием для LLM</div>
      </div>
      <div class="lo-block lo-block--retriever">
        <div class="lo-block-icon">🔎</div>
        <div class="lo-block-name">Retriever</div>
        <div class="lo-block-desc">достаёт документы<br/>из vector store</div>
      </div>
      <div class="lo-block lo-block--memory">
        <div class="lo-block-icon">💾</div>
        <div class="lo-block-name">Memory</div>
        <div class="lo-block-desc">история диалога<br/>между вызовами</div>
      </div>
    </div>

    <!-- Unifying Runnable -->
    <div class="lo-runnable" :class="{ 'lo-on': showRunnable }">
      <div class="lo-runnable-arrow">↑ все реализуют единый интерфейс ↑</div>
      <div class="lo-runnable-box">
        <span class="lo-runnable-icon">⚡</span>
        <span class="lo-runnable-name">Runnable</span>
        <span class="lo-runnable-tag">общий интерфейс — позволяет соединять блоки через <code>|</code></span>
      </div>
    </div>

  </div>
</template>

<style scoped>
.lo-root {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 26px 32px 16px;
  box-sizing: border-box;
  gap: 6px;
}

.lo-title {
  font-size: 1.15em;
  font-weight: 700;
  color: #93c5fd;
  letter-spacing: 0.02em;
  align-self: flex-start;
}

.lo-sub {
  font-size: 0.78em;
  color: #64748b;
  font-style: italic;
  align-self: flex-start;
  margin-bottom: 8px;
}

/* Blocks row */
.lo-blocks {
  display: flex;
  gap: 16px;
  margin-top: 14px;
  opacity: 0;
  transform: translateY(10px);
  transition: opacity 0.4s, transform 0.4s;
}

.lo-blocks.lo-on {
  opacity: 1;
  transform: translateY(0);
}

.lo-block {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  padding: 14px 16px 12px;
  border-radius: 12px;
  min-width: 130px;
  background: rgba(71, 85, 105, 0.10);
  border: 1.5px solid;
}

.lo-block--agent     { border-color: #3b82f6; background: rgba(37, 99, 235, 0.10); }
.lo-block--tool      { border-color: #f59e0b; background: rgba(245, 158, 11, 0.10); }
.lo-block--retriever { border-color: #10b981; background: rgba(16, 185, 129, 0.10); }
.lo-block--memory    { border-color: #a855f7; background: rgba(168, 85, 247, 0.10); }

.lo-block-icon {
  font-size: 1.6em;
  line-height: 1;
}

.lo-block-name {
  font-weight: 700;
  font-size: 0.95em;
  color: #e2e8f0;
}

.lo-block-desc {
  font-size: 0.68em;
  color: #94a3b8;
  text-align: center;
  line-height: 1.4;
}

/* Runnable */
.lo-runnable {
  margin-top: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  opacity: 0;
  transform: translateY(8px);
  transition: opacity 0.4s, transform 0.4s;
}

.lo-runnable.lo-on {
  opacity: 1;
  transform: translateY(0);
}

.lo-runnable-arrow {
  font-size: 0.7em;
  color: #475569;
  letter-spacing: 0.04em;
}

.lo-runnable-box {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 22px;
  background: rgba(37, 99, 235, 0.18);
  border: 2px solid #3b82f6;
  border-radius: 999px;
  box-shadow: 0 0 22px rgba(59, 130, 246, 0.26);
}

.lo-runnable-icon { font-size: 1.1em; }

.lo-runnable-name {
  font-weight: 700;
  font-size: 1em;
  color: #93c5fd;
  letter-spacing: 0.02em;
}

.lo-runnable-tag {
  font-size: 0.72em;
  color: #94a3b8;
  font-style: italic;
}

.lo-runnable-tag code {
  font-family: 'JetBrains Mono', monospace;
  background: rgba(15, 23, 42, 0.5);
  padding: 1px 6px;
  border-radius: 4px;
  color: #fbbf24;
  font-style: normal;
}
</style>
