<script setup>
/**
 * BootstrapFileSlide — reusable slide template for Bootstrap file explanations.
 * Renders: filename header, 2-line code example, list of content-type blocks.
 *
 * Props:
 *   filename  — file name, e.g. "SOUL.md"
 *   desc      — one-line description shown after the dash
 *   example   — short code preview (use \n for line breaks)
 *   blocks    — array of { label: string, color: string (hex) }
 */
defineProps({
  filename: { type: String, required: true },
  desc:     { type: String, required: true },
  example:  { type: String, required: true },
  blocks:   { type: Array,  required: true },
})
</script>

<template>
  <div class="bfs-root">

    <!-- ── Header: tag + filename + description ── -->
    <div class="bfs-header">
      <span class="bfs-tag">Bootstrap</span>
      <div class="bfs-title-row">
        <code class="bfs-filename">{{ filename }}</code>
        <span class="bfs-desc">— {{ desc }}</span>
      </div>
    </div>

    <!-- ── Body: two equal columns ── -->
    <div class="bfs-body">

      <!-- Left: code example -->
      <div class="bfs-col">
        <div class="bfs-label">Пример</div>
        <div class="bfs-code-wrap">
          <pre class="bfs-code">{{ example }}</pre>
        </div>
      </div>

      <!-- Right: content-type blocks -->
      <div class="bfs-col">
        <div class="bfs-label">Что содержит</div>
        <div class="bfs-blocks">
          <div
            v-for="b in blocks"
            :key="b.label"
            class="bfs-block"
            :style="{ '--c': b.color }"
          >{{ b.label }}</div>
        </div>
      </div>

    </div>
  </div>
</template>

<style scoped>
/* ── Root ────────────────────────────────────── */
.bfs-root {
  display: flex;
  flex-direction: column;
  gap: 22px;
  padding: 30px 38px;
  height: 100%;
  box-sizing: border-box;
}

/* ── Header ──────────────────────────────────── */
.bfs-header { flex-shrink: 0; }

.bfs-tag {
  display: block;
  font-size: 0.58em;
  letter-spacing: 0.2em;
  text-transform: uppercase;
  color: #475569;
  font-weight: 700;
  margin-bottom: 5px;
}

.bfs-title-row {
  display: flex;
  align-items: baseline;
  gap: 14px;
  flex-wrap: wrap;
}

/* Monospace filename — large, crisp */
.bfs-filename {
  font-family: 'JetBrains Mono', 'Fira Code', ui-monospace, monospace;
  font-size: 1.95em;
  font-weight: 800;
  color: #e2e8f0;
  letter-spacing: -0.01em;
  background: none;
  border: none;
  padding: 0;
}

.bfs-desc {
  font-size: 0.95em;
  color: #64748b;
  font-weight: 400;
}

/* ── Two-column body ─────────────────────────── */
.bfs-body {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 26px;
  flex: 1;
  min-height: 0;
}

.bfs-col {
  display: flex;
  flex-direction: column;
  gap: 10px;
  min-height: 0;
}

.bfs-label {
  font-size: 0.58em;
  letter-spacing: 0.18em;
  text-transform: uppercase;
  color: #475569;
  font-weight: 700;
  flex-shrink: 0;
}

/* ── Code block ──────────────────────────────── */
.bfs-code-wrap {
  flex: 1;
  background: rgba(2, 6, 23, 0.55);
  border: 1px solid rgba(51, 65, 85, 0.65);
  border-radius: 10px;
  padding: 18px 20px;
  overflow: hidden;
}

.bfs-code {
  font-family: 'JetBrains Mono', 'Fira Code', ui-monospace, monospace;
  font-size: 0.78em;
  color: #94a3b8;
  margin: 0;
  line-height: 1.8;
  white-space: pre-wrap;
}

/* ── Content blocks ──────────────────────────── */
.bfs-blocks {
  display: flex;
  flex-direction: column;
  gap: 9px;
  flex: 1;
  min-height: 0;
  overflow: hidden;
}

/* Each block stretches to fill available height equally */
.bfs-block {
  flex: 1;
  display: flex;
  align-items: center;
  padding: 0 18px;
  border-radius: 10px;
  font-size: 0.88em;
  font-weight: 600;
  letter-spacing: 0.01em;
  user-select: none;
  /* color-mix() lightens/saturates the hex for readable text */
  background: color-mix(in srgb, var(--c) 10%, transparent);
  border: 1.5px solid color-mix(in srgb, var(--c) 40%, transparent);
  color: color-mix(in srgb, var(--c) 90%, white 10%);
}
</style>
