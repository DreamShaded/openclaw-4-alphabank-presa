<script setup>
import { useNav } from '@slidev/client'
import { computed } from 'vue'

const { clicks } = useNav()

const show1 = computed(() => clicks.value >= 1) // simple skills
const show2 = computed(() => clicks.value >= 2) // RAG
const show3 = computed(() => clicks.value >= 3) // MCP
const show4 = computed(() => clicks.value >= 4) // LangChain
const show5 = computed(() => clicks.value >= 5) // final — complex skills
</script>

<template>
  <div class="tk-root">

    <div class="tk-title">Что у нас теперь в руках</div>

    <div class="tk-list">

      <div class="tk-item" :class="{ 'tk-on': show1 }">
        <div class="tk-check">✓</div>
        <div class="tk-text">Научились делать <b>простые скиллы</b> — markdown-файл с frontmatter</div>
      </div>

      <div class="tk-item" :class="{ 'tk-on': show2 }">
        <div class="tk-check">✓</div>
        <div class="tk-text">Разобрались с <b>RAG</b> — эмбеддинги, hybrid-поиск, Qdrant</div>
      </div>

      <div class="tk-item" :class="{ 'tk-on': show3 }">
        <div class="tk-check">✓</div>
        <div class="tk-text">Разобрались с <b>MCP</b> — stateful сессия, JSON-RPC, registerTool</div>
      </div>

      <div class="tk-item" :class="{ 'tk-on': show4 }">
        <div class="tk-check">✓</div>
        <div class="tk-text">Разобрались с <b>LangChain</b> — Runnable, LCEL, LangGraph для FSM</div>
      </div>

    </div>

    <!-- Final big takeaway -->
    <div class="tk-final" :class="{ 'tk-on': show5 }">
      <div class="tk-final-icon">🚀</div>
      <div class="tk-final-text">
        <span class="tk-final-lead">Можем делать сложные скиллы —</span>
        <span class="tk-final-body">сцепляя LLM и API нужного сервиса</span>
      </div>
    </div>

  </div>
</template>

<style scoped>
.tk-root {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 36px 56px 28px;
  box-sizing: border-box;
  gap: 22px;
}

.tk-title {
  font-size: 1.4em;
  font-weight: 700;
  color: #93c5fd;
  letter-spacing: 0.02em;
  align-self: flex-start;
}

/* Checklist */
.tk-list {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  gap: 12px;
  width: 100%;
  max-width: 720px;
}

.tk-item {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 10px 18px;
  border-radius: 10px;
  background: rgba(71, 85, 105, 0.10);
  border: 1px solid #334155;
  opacity: 0;
  transform: translateX(-12px);
  transition: opacity 0.35s, transform 0.35s, border-color 0.35s, background 0.35s;
}

.tk-item.tk-on {
  opacity: 1;
  transform: translateX(0);
  background: rgba(37, 99, 235, 0.12);
  border-color: #3b82f6;
}

.tk-check {
  flex-shrink: 0;
  width: 30px;
  height: 30px;
  border-radius: 50%;
  background: #3b82f6;
  color: #fff;
  font-weight: 700;
  font-size: 1.05em;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 0 12px rgba(59, 130, 246, 0.4);
}

.tk-text {
  font-size: 0.95em;
  color: #cbd5e1;
  line-height: 1.4;
}

.tk-text b {
  color: #93c5fd;
  font-weight: 700;
}

/* Final takeaway */
.tk-final {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 18px 28px;
  border-radius: 14px;
  background: linear-gradient(135deg, rgba(37, 99, 235, 0.22), rgba(168, 85, 247, 0.18));
  border: 2px solid #3b82f6;
  box-shadow: 0 0 32px rgba(59, 130, 246, 0.32);
  width: 100%;
  max-width: 720px;
  opacity: 0;
  transform: translateY(12px) scale(0.96);
  transition: opacity 0.45s, transform 0.45s;
}

.tk-final.tk-on {
  opacity: 1;
  transform: translateY(0) scale(1);
}

.tk-final-icon {
  font-size: 1.8em;
  line-height: 1;
}

.tk-final-text {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.tk-final-lead {
  font-size: 1.05em;
  font-weight: 700;
  color: #93c5fd;
  letter-spacing: 0.01em;
}

.tk-final-body {
  font-size: 0.85em;
  color: #cbd5e1;
}
</style>
