<script setup>
import { useNav } from '@slidev/client'
import { computed } from 'vue'

const { clicks } = useNav()

const showFlow = computed(() => clicks.value >= 1)
const showCode = computed(() => clicks.value >= 2)
const showSwap = computed(() => clicks.value >= 3)
</script>

<template>
  <div class="lc-root">

    <div class="lc-title">LCEL — собираем цепочку через <code>|</code></div>
    <div class="lc-sub">LangChain Expression Language · pipe-оператор как в shell</div>

    <!-- Pipe-flow blocks -->
    <div class="lc-flow" :class="{ 'lc-on': showFlow }">
      <div class="lc-block lc-block--prompt">
        <div class="lc-block-tag">Runnable</div>
        <div class="lc-block-icon">📝</div>
        <div class="lc-block-name">Prompt</div>
        <div class="lc-block-hint">шаблон с переменными</div>
      </div>

      <div class="lc-pipe">|</div>

      <div class="lc-block lc-block--llm">
        <div class="lc-block-tag">Runnable</div>
        <div class="lc-block-icon">🤖</div>
        <div class="lc-block-name">LLM</div>
        <div class="lc-block-hint">любая модель</div>
      </div>

      <div class="lc-pipe">|</div>

      <div class="lc-block lc-block--parser">
        <div class="lc-block-tag">Runnable</div>
        <div class="lc-block-icon">🧩</div>
        <div class="lc-block-name">Parser</div>
        <div class="lc-block-hint">текст → объект</div>
      </div>
    </div>

    <!-- Code example -->
    <div class="lc-code-wrap" :class="{ 'lc-on': showCode }">
      <pre class="lc-code">
chain = prompt <span class="lc-pipe-in">|</span> llm <span class="lc-pipe-in">|</span> parser

result = chain.invoke({<span class="lc-str">"topic"</span>: <span class="lc-str">"MCP"</span>})</pre>
    </div>

    <!-- Swap hint -->
    <div class="lc-swap" :class="{ 'lc-on': showSwap }">
      <div class="lc-swap-line">
        <span class="lc-swap-icon">⇄</span>
        <span>любой блок можно подменить — интерфейс <code>Runnable</code> один и тот же</span>
      </div>
      <div class="lc-swap-line">
        <span class="lc-swap-icon">📦</span>
        <span>добавить retriever, memory, tool — просто ещё один <code>|</code></span>
      </div>
    </div>

  </div>
</template>

<style scoped>
.lc-root {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 24px 28px 14px;
  box-sizing: border-box;
  gap: 6px;
}

.lc-title {
  font-size: 1.15em;
  font-weight: 700;
  color: #93c5fd;
  letter-spacing: 0.02em;
  align-self: flex-start;
}

.lc-title code {
  font-family: 'JetBrains Mono', monospace;
  background: rgba(15, 23, 42, 0.6);
  padding: 2px 8px;
  border-radius: 4px;
  color: #fbbf24;
  font-size: 0.85em;
}

.lc-sub {
  font-size: 0.78em;
  color: #64748b;
  font-style: italic;
  align-self: flex-start;
  margin-bottom: 10px;
}

/* Flow with pipes */
.lc-flow {
  margin-top: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  opacity: 0;
  transform: translateY(8px);
  transition: opacity 0.4s, transform 0.4s;
}

.lc-flow.lc-on {
  opacity: 1;
  transform: translateY(0);
}

.lc-block {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  padding: 16px 22px 12px;
  border-radius: 12px;
  min-width: 130px;
  background: rgba(71, 85, 105, 0.12);
  border: 1.5px solid;
}

.lc-block--prompt { border-color: #10b981; background: rgba(16, 185, 129, 0.10); }
.lc-block--llm    { border-color: #3b82f6; background: rgba(37, 99, 235, 0.12); }
.lc-block--parser { border-color: #f59e0b; background: rgba(245, 158, 11, 0.10); }

.lc-block-tag {
  position: absolute;
  top: -10px;
  background: #1e293b;
  border: 1px solid #475569;
  color: #94a3b8;
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.55em;
  font-weight: 600;
  padding: 1px 8px;
  border-radius: 999px;
  letter-spacing: 0.06em;
}

.lc-block-icon {
  font-size: 1.5em;
  line-height: 1;
}

.lc-block-name {
  font-weight: 700;
  font-size: 0.95em;
  color: #e2e8f0;
}

.lc-block-hint {
  font-size: 0.65em;
  color: #94a3b8;
  text-align: center;
  font-style: italic;
}

.lc-pipe {
  font-family: 'JetBrains Mono', monospace;
  font-size: 1.8em;
  font-weight: 700;
  color: #fbbf24;
  line-height: 1;
}

/* Code block */
.lc-code-wrap {
  margin-top: 16px;
  opacity: 0;
  transform: translateY(8px);
  transition: opacity 0.4s, transform 0.4s;
}

.lc-code-wrap.lc-on {
  opacity: 1;
  transform: translateY(0);
}

.lc-code {
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.78em;
  color: #cbd5e1;
  background: rgba(15, 23, 42, 0.65);
  border: 1px solid #334155;
  border-radius: 8px;
  padding: 14px 22px;
  margin: 0;
  line-height: 1.7;
  white-space: pre;
}

.lc-com       { color: #64748b; font-style: italic; }
.lc-pipe-in   { color: #fbbf24; font-weight: 700; }
.lc-str       { color: #6ee7b7; }

/* Swap hints */
.lc-swap {
  margin-top: 14px;
  display: flex;
  flex-direction: column;
  gap: 6px;
  opacity: 0;
  transform: translateY(6px);
  transition: opacity 0.4s, transform 0.4s;
}

.lc-swap.lc-on {
  opacity: 1;
  transform: translateY(0);
}

.lc-swap-line {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 0.78em;
  color: #94a3b8;
}

.lc-swap-icon {
  font-size: 1em;
  color: #60a5fa;
  width: 20px;
}

.lc-swap-line code {
  font-family: 'JetBrains Mono', monospace;
  background: rgba(15, 23, 42, 0.5);
  padding: 1px 6px;
  border-radius: 4px;
  color: #fbbf24;
  font-size: 0.85em;
}
</style>
