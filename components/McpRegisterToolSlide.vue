<script setup>
import { useNav } from '@slidev/client'
import { computed } from 'vue'

const { clicks } = useNav()

const hi1 = computed(() => clicks.value >= 1) // name
const hi2 = computed(() => clicks.value >= 2) // description
const hi3 = computed(() => clicks.value >= 3) // inputSchema
const hi4 = computed(() => clicks.value >= 4) // handler
</script>

<template>
  <div class="mr-root">

    <div class="mr-title">Как навык появляется в агенте — <code>registerTool</code></div>
    <div class="mr-sub">обёртка = тонкий MCP-сервер на @modelcontextprotocol/sdk</div>

    <div class="mr-layout">

      <!-- Left: code -->
      <div class="mr-code">
<pre><span class="mr-fn">mcp.registerTool</span>(
  <span class="mr-line" :class="{ 'mr-on': hi1 }"><span class="mr-str">"plan_task"</span>,                          <span class="mr-com">// имя — видит агент</span></span>
  {
    <span class="mr-line" :class="{ 'mr-on': hi2 }">description: <span class="mr-str">"что делает,</span>
                  <span class="mr-str">когда звать, что вернёт..."</span>,</span>
    <span class="mr-line" :class="{ 'mr-on': hi3 }">inputSchema: { rawRequest: z.string()...
                  deadlineIso: z.string().datetime() }</span>,
  },
  <span class="mr-line" :class="{ 'mr-on': hi4 }"><span class="mr-kw">async</span> (args) =&gt; {              <span class="mr-com">// обработчик</span>
    <span class="mr-kw">const</span> r = <span class="mr-kw">await</span> postJson(<span class="mr-str">"/plan"</span>, args)
    <span class="mr-kw">return</span> asContent(r.body)        <span class="mr-com">// { content, isError? }</span>
  }</span>,
)</pre>
      </div>

      <!-- Right: 4 annotation cards -->
      <div class="mr-notes">
        <div class="mr-note" :class="{ 'mr-on': hi1 }">
          <div class="mr-note-num">1</div>
          <div class="mr-note-body">
            <div class="mr-note-name">name</div>
            <div class="mr-note-text">короткий id навыка</div>
          </div>
        </div>

        <div class="mr-note mr-note--key" :class="{ 'mr-on': hi2 }">
          <div class="mr-note-num">2</div>
          <div class="mr-note-body">
            <div class="mr-note-name">description</div>
            <div class="mr-note-text">это <b>промпт для LLM</b>, не для человека —<br/>агент по нему выбирает инструмент</div>
          </div>
        </div>

        <div class="mr-note" :class="{ 'mr-on': hi3 }">
          <div class="mr-note-num">3</div>
          <div class="mr-note-body">
            <div class="mr-note-name">inputSchema · zod</div>
            <div class="mr-note-text">SDK валидирует аргументы<br/>и публикует схему агенту</div>
          </div>
        </div>

        <div class="mr-note" :class="{ 'mr-on': hi4 }">
          <div class="mr-note-num">4</div>
          <div class="mr-note-body">
            <div class="mr-note-name">handler</div>
            <div class="mr-note-text">проксирует во внутренний HTTP-сервис,<br/>возвращает <code>content[]</code> + <code>isError?</code></div>
          </div>
        </div>
      </div>

    </div>

    <div class="mr-foot">всё остальное — транспорт, сессии, /mcp endpoint — SDK берёт на себя</div>

  </div>
</template>

<style scoped>
.mr-root {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 22px 28px 14px;
  box-sizing: border-box;
  gap: 6px;
}

.mr-title {
  font-size: 1.1em;
  font-weight: 700;
  color: #93c5fd;
  letter-spacing: 0.02em;
  align-self: flex-start;
}

.mr-title code {
  font-family: 'JetBrains Mono', monospace;
  background: rgba(15, 23, 42, 0.6);
  padding: 2px 8px;
  border-radius: 4px;
  color: #fbbf24;
  font-size: 0.85em;
}

.mr-sub {
  font-size: 0.74em;
  color: #64748b;
  font-style: italic;
  align-self: flex-start;
  margin-bottom: 8px;
}

.mr-layout {
  flex: 1;
  display: flex;
  align-items: stretch;
  justify-content: center;
  gap: 18px;
  width: 100%;
}

/* Code panel */
.mr-code {
  flex: 1.4;
  background: rgba(15, 23, 42, 0.65);
  border: 1px solid #334155;
  border-radius: 10px;
  padding: 14px 18px;
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.66em;
  color: #cbd5e1;
  line-height: 1.65;
  overflow: hidden;
  display: flex;
  align-items: center;
}

.mr-code pre {
  margin: 0;
  white-space: pre;
}

.mr-line {
  display: inline-block;
  width: 100%;
  padding: 0 6px;
  margin: 0 -6px;
  border-radius: 4px;
  transition: background 0.3s, color 0.3s;
  color: #64748b;
}

.mr-line.mr-on {
  background: rgba(59, 130, 246, 0.14);
  color: #e2e8f0;
}

.mr-fn  { color: #60a5fa; font-weight: 700; }
.mr-str { color: #fbbf24; }
.mr-kw  { color: #c084fc; }
.mr-com { color: #475569; font-style: italic; }

/* Notes panel */
.mr-notes {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 8px;
  max-width: 280px;
}

.mr-note {
  display: flex;
  align-items: flex-start;
  gap: 10px;
  padding: 8px 12px;
  border-radius: 8px;
  background: rgba(71, 85, 105, 0.10);
  border: 1px solid #334155;
  opacity: 0.35;
  transform: translateX(-4px);
  transition: opacity 0.35s, transform 0.35s, border-color 0.35s, background 0.35s;
}

.mr-note.mr-on {
  opacity: 1;
  transform: translateX(0);
  border-color: #3b82f6;
  background: rgba(37, 99, 235, 0.12);
}

.mr-note--key.mr-on {
  border-color: #fbbf24;
  background: rgba(245, 158, 11, 0.12);
  box-shadow: 0 0 14px rgba(245, 158, 11, 0.18);
}

.mr-note-num {
  flex-shrink: 0;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #3b82f6;
  color: #fff;
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.7em;
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: center;
}

.mr-note--key.mr-on .mr-note-num {
  background: #f59e0b;
}

.mr-note-body {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.mr-note-name {
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.78em;
  font-weight: 700;
  color: #e2e8f0;
}

.mr-note-text {
  font-size: 0.7em;
  color: #94a3b8;
  line-height: 1.4;
}

.mr-note-text b {
  color: #fbbf24;
  font-weight: 700;
}

.mr-note-text code {
  font-family: 'JetBrains Mono', monospace;
  background: rgba(15, 23, 42, 0.5);
  padding: 1px 5px;
  border-radius: 3px;
  color: #93c5fd;
  font-size: 0.9em;
}

.mr-foot {
  font-size: 0.72em;
  color: #475569;
  font-style: italic;
  text-align: center;
}
</style>
