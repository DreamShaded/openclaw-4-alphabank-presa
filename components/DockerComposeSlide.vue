<script setup>
/**
 * DockerComposeSlide — shows docker-compose.yml + .env setup for OpenClaw launch.
 * Highlights volume mounts (config/, workspace/) and model key (Kimi K2.6).
 * Static content — no props needed.
 */

const composeLines = [
  { text: 'services:',                                      cls: '' },
  { text: '  openclaw:',                                    cls: '' },
  { text: '    image: ghcr.io/openclaw/openclaw:${TAG}',   cls: 'dim' },
  { text: '    ports:',                                     cls: 'dim' },
  { text: '      - "127.0.0.1:18789:18789"',               cls: 'dim' },
  { text: '    volumes:',                                   cls: 'key' },
  { text: '      - ./config:/home/node/.openclaw',         cls: 'vol' },
  { text: '      - ./workspace:/…/.openclaw/workspace',    cls: 'vol' },
  { text: '    env_file: [.env]',                          cls: 'env' },
  { text: '    init: true',                                cls: 'dim' },
]

const envLines = [
  { text: 'OPENCLAW_TAG=latest',          cls: 'dim' },
  { text: 'OPENCLAW_GATEWAY_PORT=18789',  cls: 'dim' },
  { text: 'OPENCLAW_GATEWAY_TOKEN=',      cls: 'dim' },
  { text: '',                             cls: '' },
  { text: '# Kimi K2.6 — primary',       cls: 'comment' },
  { text: 'MOONSHOT_API_KEY=sk-…',        cls: 'model' },
  { text: '',                             cls: '' },
  { text: '# опционально',               cls: 'comment' },
  { text: 'ANTHROPIC_API_KEY=',          cls: 'dim' },
  { text: 'OLLAMA_API_KEY=ollama-local', cls: 'dim' },
]
</script>

<template>
  <div class="dcs-root">

    <!-- ── Header ── -->
    <div class="dcs-header">
      <span class="dcs-tag">Запуск</span>
      <div class="dcs-title-row">
        <code class="dcs-cmd">docker compose up -d</code>
        <span class="dcs-sub">— два маунта, один .env</span>
      </div>
    </div>

    <!-- ── Two-column body ── -->
    <div class="dcs-body">

      <!-- Left: docker-compose.yml -->
      <div class="dcs-col">
        <div class="dcs-label">docker-compose.yml</div>
        <div class="dcs-code-wrap">
          <code class="dcs-code">
            <span
              v-for="(line, i) in composeLines"
              :key="i"
              :class="['dcs-line', 'ln-' + line.cls]"
            >{{ line.text }}</span>
          </code>
        </div>
        <div class="dcs-badges">
          <div class="dcs-badge badge-teal">📁 config/ → настройки агента</div>
          <div class="dcs-badge badge-teal">📂 workspace/ → навыки и файлы</div>
        </div>
      </div>

      <!-- Right: .env -->
      <div class="dcs-col">
        <div class="dcs-label">.env</div>
        <div class="dcs-code-wrap">
          <code class="dcs-code">
            <span
              v-for="(line, i) in envLines"
              :key="i"
              :class="['dcs-line', 'ln-' + line.cls]"
            >{{ line.text }}</span>
          </code>
        </div>
        <div class="dcs-badges">
          <div class="dcs-badge badge-purple">🔑 Kimi K2.6 → Moonshot API</div>
        </div>
      </div>

    </div>
  </div>
</template>

<style scoped>
/* ── Root ───────────────────────────────────── */
.dcs-root {
  display: flex;
  flex-direction: column;
  gap: 20px;
  padding: 30px 38px;
  height: 100%;
  box-sizing: border-box;
}

/* ── Header ─────────────────────────────────── */
.dcs-header { flex-shrink: 0; }

.dcs-tag {
  display: block;
  font-size: 0.58em;
  letter-spacing: 0.2em;
  text-transform: uppercase;
  color: #475569;
  font-weight: 700;
  margin-bottom: 5px;
}

.dcs-title-row {
  display: flex;
  align-items: baseline;
  gap: 14px;
  flex-wrap: wrap;
}

.dcs-cmd {
  font-family: 'JetBrains Mono', 'Fira Code', ui-monospace, monospace;
  font-size: 1.95em;
  font-weight: 800;
  color: #e2e8f0;
  letter-spacing: -0.01em;
  background: none;
  border: none;
  padding: 0;
}

.dcs-sub {
  font-size: 0.95em;
  color: #64748b;
  font-weight: 400;
}

/* ── Two-column grid ─────────────────────────── */
.dcs-body {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 24px;
  flex: 1;
  min-height: 0;
}

.dcs-col {
  display: flex;
  flex-direction: column;
  gap: 10px;
  min-height: 0;
}

.dcs-label {
  font-size: 0.58em;
  letter-spacing: 0.18em;
  text-transform: uppercase;
  color: #475569;
  font-weight: 700;
  flex-shrink: 0;
}

/* ── Code block ──────────────────────────────── */
.dcs-code-wrap {
  flex: 1;
  background: rgba(2, 6, 23, 0.55);
  border: 1px solid rgba(51, 65, 85, 0.65);
  border-radius: 10px;
  padding: 16px 18px;
  overflow: hidden;
  min-height: 0;
}

.dcs-code {
  font-family: 'JetBrains Mono', 'Fira Code', ui-monospace, monospace;
  font-size: 0.72em;
  display: block;
  line-height: 1.75;
}

/* Each line rendered as a block — preserves indentation */
.dcs-line {
  display: block;
  white-space: pre;
  color: #94a3b8;
}

/* Line color variants */
.ln-    { color: #cbd5e1; }
.ln-dim { color: #475569; }

/* volumes: label — sky blue */
.ln-key { color: #38bdf8; }

/* volume paths — teal, subtle highlight */
.ln-vol {
  color: #2dd4bf;
  background: rgba(45, 212, 191, 0.07);
  border-radius: 2px;
}

/* env_file — amber */
.ln-env { color: #fbbf24; }

/* # comments — slate italic */
.ln-comment {
  color: #64748b;
  font-style: italic;
}

/* MOONSHOT_API_KEY — violet, highlighted */
.ln-model {
  color: #a78bfa;
  background: rgba(167, 139, 250, 0.1);
  border-radius: 2px;
}

/* ── Badge row ───────────────────────────────── */
.dcs-badges {
  display: flex;
  flex-direction: column;
  gap: 7px;
  flex-shrink: 0;
}

.dcs-badge {
  padding: 7px 14px;
  border-radius: 8px;
  font-size: 0.78em;
  font-weight: 600;
  letter-spacing: 0.01em;
  user-select: none;
}

/* Teal badge — for volume annotations */
.badge-teal {
  background: rgba(45, 212, 191, 0.1);
  border: 1.5px solid rgba(45, 212, 191, 0.35);
  color: #5eead4;
}

/* Purple badge — for model annotation */
.badge-purple {
  background: rgba(167, 139, 250, 0.1);
  border: 1.5px solid rgba(167, 139, 250, 0.35);
  color: #c4b5fd;
}
</style>
