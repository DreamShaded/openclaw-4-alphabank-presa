<script setup>
import { useNav } from '@slidev/client'
import { computed } from 'vue'

const { clicks } = useNav()

const hi1 = computed(() => clicks.value >= 1) // frontmatter
const hi2 = computed(() => clicks.value >= 2) // body
</script>

<template>
  <div class="ne-root">

    <div class="ne-title">Заметка — это просто Markdown</div>

    <div class="ne-layout">

      <!-- Note card -->
      <div class="ne-note">
        <div class="ne-filename">📄 2026-05-12-mcp-server.md</div>

        <pre class="ne-code"><span class="ne-block" :class="{ 'ne-on': hi1 }"><span class="ne-fence">---</span>
<span class="ne-key">title</span>: Как поднять MCP-сервер на Fastify
<span class="ne-key">tags</span>: [mcp, fastify, typescript]
<span class="ne-key">date</span>: 2026-05-12
<span class="ne-key">project</span>: openclaw
<span class="ne-fence">---</span></span>

<span class="ne-block" :class="{ 'ne-on': hi2 }"><span class="ne-h1"># Как поднять MCP-сервер на Fastify</span>

MCP — stateful протокол поверх HTTP+SSE.
Сервер держит долгоживущее соединение и пушит
notifications обратно клиенту.

<span class="ne-h2">## Ключевые шаги</span>
- Регистрируем endpoint <span class="ne-inline">/mcp</span> с двумя методами
- POST принимает JSON-RPC сообщения
- GET держит SSE-стрим открытым
- `initialize` → `discovery` → `operation`...</span></pre>
      </div>

      <!-- Annotations -->
      <div class="ne-annotations">
        <div class="ne-ann" :class="{ 'ne-ann-on': hi1 }">
          <div class="ne-ann-badge ne-ann-badge--meta">frontmatter</div>
          <div class="ne-ann-text">
            метаданные: теги, дата, проект<br/>
            <span class="ne-ann-hint">→ для фильтрации в поиске</span>
          </div>
        </div>
        <div class="ne-ann" :class="{ 'ne-ann-on': hi2 }">
          <div class="ne-ann-badge ne-ann-badge--body">тело</div>
          <div class="ne-ann-text">
            обычный Markdown<br/>
            <span class="ne-ann-hint">→ для семантического поиска</span>
          </div>
        </div>
      </div>

    </div>

    <div class="ne-foot">тысячи таких файлов = ваша личная база знаний</div>

  </div>
</template>

<style scoped>
.ne-root {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 24px 32px 16px;
  box-sizing: border-box;
  gap: 14px;
}

.ne-title {
  font-size: 1.1em;
  font-weight: 700;
  color: #93c5fd;
  letter-spacing: 0.02em;
  align-self: flex-start;
}

.ne-layout {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 24px;
  width: 100%;
}

/* Note card */
.ne-note {
  background: rgba(15, 23, 42, 0.6);
  border: 1.5px solid #334155;
  border-radius: 10px;
  padding: 10px 14px 14px;
  min-width: 520px;
  box-shadow: 0 4px 18px rgba(0, 0, 0, 0.3);
}

.ne-filename {
  font-size: 0.7em;
  color: #64748b;
  margin-bottom: 8px;
  padding-bottom: 6px;
  border-bottom: 1px solid #1e293b;
  font-family: 'JetBrains Mono', monospace;
}

.ne-code {
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.66em;
  color: #94a3b8;
  margin: 0;
  line-height: 1.55;
  white-space: pre;
}

.ne-block {
  display: inline-block;
  width: 100%;
  padding: 2px 6px;
  margin: 0 -6px;
  border-radius: 4px;
  transition: background 0.3s, color 0.3s;
}

.ne-block.ne-on {
  background: rgba(59, 130, 246, 0.12);
  color: #cbd5e1;
}

.ne-fence  { color: #64748b; }
.ne-key    { color: #60a5fa; }
.ne-h1     { color: #93c5fd; font-weight: 700; }
.ne-h2     { color: #93c5fd; font-weight: 600; }
.ne-inline { color: #fbbf24; }

/* Annotations */
.ne-annotations {
  display: flex;
  flex-direction: column;
  gap: 18px;
  max-width: 220px;
}

.ne-ann {
  opacity: 0.3;
  transform: translateX(-4px);
  transition: opacity 0.3s, transform 0.3s;
}

.ne-ann.ne-ann-on {
  opacity: 1;
  transform: translateX(0);
}

.ne-ann-badge {
  display: inline-block;
  padding: 3px 10px;
  border-radius: 6px;
  font-size: 0.7em;
  font-weight: 700;
  letter-spacing: 0.04em;
  margin-bottom: 6px;
}

.ne-ann-badge--meta {
  background: rgba(96, 165, 250, 0.2);
  border: 1px solid #60a5fa;
  color: #93c5fd;
}

.ne-ann-badge--body {
  background: rgba(168, 85, 247, 0.18);
  border: 1px solid #a855f7;
  color: #c4b5fd;
}

.ne-ann-text {
  font-size: 0.78em;
  color: #94a3b8;
  line-height: 1.45;
}

.ne-ann-hint {
  font-size: 0.85em;
  color: #64748b;
  font-style: italic;
}

.ne-foot {
  font-size: 0.74em;
  color: #475569;
  font-style: italic;
}
</style>
