<script setup>
/**
 * SkillPipelineSlide — how OpenClaw auto-discovers skills on session start.
 * Shows: pipeline log → frontmatter in system prompt → lazy body load → condition table.
 */

const frontmatterLines = [
  { text: 'name: no-junk',            key: true  },
  { text: 'description: Отговорить',  key: true  },
  { text: '  от джанка.',             key: false },
  { text: '  Выяснить: голод?',       key: false },
]

const bodyLines = [
  { text: '## Когда использовать' },
  { text: '«хочу энергетик» → …'  },
  { text: '## Протокол'           },
  { text: 'Шаг 1. Acknowledge…'   },
]

const conditions = [
  { icon: '✓', cond: 'workspace/skills/<name>/SKILL.md',    effect: 'Подхватится автоматически на следующей сессии',    v: 'ok'   },
  { icon: '✓', cond: 'Frontmatter name + description валидны', effect: 'Попадёт в системный промпт как каталог',         v: 'ok'   },
]
</script>

<template>
  <div class="sps-root">

    <!-- Header -->
    <div class="sps-header">
      <span class="sps-tag">Skills Pipeline</span>
      <div class="sps-title-row">
        <span class="sps-title">Автообнаружение навыков</span>
        <span class="sps-sub">— без перезагрузки контейнера</span>
      </div>
    </div>

    <!-- Pipeline log strip -->
    <div class="sps-log"><span class="log-dim">[agent/embedded] prep stages: …</span><span class="log-hi">skills:0ms@1ms</span><span class="log-dim">,core-plugin-tools:…</span></div>

    <!-- Two panels: system prompt vs lazy body -->
    <div class="sps-mid">

      <div class="sps-panel sps-panel--catalog">
        <div class="sps-panel-lbl">📋 System Prompt · каждая сессия</div>
        <div class="sps-code-box">
          <code class="sps-code">
            <span v-for="(l, i) in frontmatterLines" :key="i" :class="['sps-ln', l.key ? 'ln-key' : 'ln-val']">{{ l.text }}</span>
          </code>
        </div>
      </div>

      <div class="sps-sep">
        <span class="sep-label">trigger</span>
        <span class="sep-arrow">→</span>
      </div>

      <div class="sps-panel sps-panel--lazy">
        <div class="sps-panel-lbl">⚡ Body навыка · лениво</div>
        <div class="sps-code-box sps-code-box--dim">
          <code class="sps-code">
            <span v-for="(l, i) in bodyLines" :key="i" class="sps-ln ln-muted">{{ l.text }}</span>
          </code>
        </div>
      </div>

    </div>

    <!-- Condition table -->
    <div class="sps-table">
      <div v-for="row in conditions" :key="row.cond" :class="['sps-row', `sps-row--${row.v}`]">
        <span class="row-icon">{{ row.icon }}</span>
        <code class="row-cond">{{ row.cond }}</code>
        <span class="row-arrow">→</span>
        <span class="row-effect">{{ row.effect }}</span>
      </div>
    </div>

  </div>
</template>

<style scoped>
.sps-root {
  display: flex;
  flex-direction: column;
  gap: 13px;
  padding: 28px 38px;
  height: 100%;
  box-sizing: border-box;
}

/* ── Header ── */
.sps-header { flex-shrink: 0; }

.sps-tag {
  display: block;
  font-size: 0.58em;
  letter-spacing: 0.2em;
  text-transform: uppercase;
  color: #475569;
  font-weight: 700;
  margin-bottom: 4px;
}

.sps-title-row { display: flex; align-items: baseline; gap: 12px; }

.sps-title {
  font-size: 1.85em;
  font-weight: 800;
  color: #e2e8f0;
  letter-spacing: -0.01em;
}

.sps-sub { font-size: 0.9em; color: #64748b; }

/* ── Log strip ── */
.sps-log {
  flex-shrink: 0;
  background: rgba(2, 6, 23, 0.7);
  border: 1px solid rgba(51, 65, 85, 0.7);
  border-radius: 8px;
  padding: 8px 16px;
  font-family: 'JetBrains Mono', 'Fira Code', ui-monospace, monospace;
  font-size: 0.71em;
  white-space: nowrap;
  overflow: hidden;
}

.log-dim { color: #475569; }

.log-hi {
  color: #06b6d4;
  font-weight: 700;
  background: rgba(6, 182, 212, 0.1);
  border-radius: 3px;
  padding: 0 3px;
}

/* ── Two panels ── */
.sps-mid {
  display: grid;
  grid-template-columns: 1fr 46px 1fr;
  align-items: center;
  flex-shrink: 0;
}

.sps-panel { display: flex; flex-direction: column; gap: 6px; }

.sps-panel-lbl {
  font-size: 0.60em;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  font-weight: 700;
}

.sps-panel--catalog .sps-panel-lbl { color: #38bdf8; }
.sps-panel--lazy    .sps-panel-lbl { color: #a78bfa; }

.sps-code-box {
  background: rgba(2, 6, 23, 0.55);
  border-radius: 10px;
  padding: 12px 16px;
  overflow: hidden;
}

.sps-panel--catalog .sps-code-box { border: 1px solid rgba(56, 189, 248, 0.3); }
.sps-code-box--dim {
  border: 1px solid rgba(167, 139, 250, 0.2);
  opacity: 0.6;
}

.sps-code {
  font-family: 'JetBrains Mono', 'Fira Code', ui-monospace, monospace;
  font-size: 0.73em;
  display: block;
  line-height: 1.7;
}

.sps-ln    { display: block; white-space: pre; }
.ln-key    { color: #38bdf8; }
.ln-val    { color: #94a3b8; }
.ln-muted  { color: #475569; }

/* Separator */
.sps-sep {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
}

.sep-label {
  font-size: 0.54em;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: #475569;
  font-weight: 600;
}

.sep-arrow { font-size: 1.4em; color: #475569; line-height: 1; }

/* ── Condition table ── */
.sps-table { display: flex; flex-direction: column; gap: 7px; flex: 1; min-height: 0; }

.sps-row {
  display: grid;
  grid-template-columns: 24px 1fr 18px 1fr;
  align-items: center;
  gap: 10px;
  padding: 7px 14px;
  border-radius: 8px;
  font-size: 0.79em;
  flex: 1;
}

.row-icon  { font-weight: 700; text-align: center; }
.row-cond  { font-family: 'JetBrains Mono', 'Fira Code', ui-monospace, monospace; font-size: 0.9em; font-weight: 600; }
.row-arrow { color: #475569; text-align: center; }
.row-effect { color: #94a3b8; }

/* Row variants */
.sps-row--ok   { background: rgba(34, 197, 94, 0.07);  border: 1px solid rgba(34, 197, 94, 0.25);  }
.sps-row--warn { background: rgba(245, 158, 11, 0.07); border: 1px solid rgba(245, 158, 11, 0.25); }
.sps-row--info { background: rgba(59, 130, 246, 0.07); border: 1px solid rgba(59, 130, 246, 0.25); }

.sps-row--ok   .row-icon { color: #22c55e; }
.sps-row--warn .row-icon { color: #f59e0b; }
.sps-row--info .row-icon { color: #64748b; }

.sps-row--ok   .row-cond { color: #86efac; }
.sps-row--warn .row-cond { color: #fcd34d; }
.sps-row--info .row-cond { color: #94a3b8; text-decoration: line-through; text-decoration-color: #475569; }
</style>
