<script setup>
import { useNav } from '@slidev/client'
import { computed } from 'vue'

const { clicks } = useNav()

const showReq    = computed(() => clicks.value >= 1)
const showResp   = computed(() => clicks.value >= 2)
const showNotify = computed(() => clicks.value >= 3)
</script>

<template>
  <div class="mms-root">

    <div class="mms-title">
      Три типа сообщений <span class="mms-title-sub">— JSON-RPC 2.0</span>
    </div>

    <div class="mms-grid">

      <!-- Request -->
      <div class="mms-card mms-card--req" :class="{ 'mms-visible': showReq }">
        <div class="mms-card-head">
          <div class="mms-card-name">Request</div>
          <div class="mms-card-dir">клиент → сервер</div>
        </div>
        <pre class="mms-code"><span class="mms-key">"id"</span>: 1,
<span class="mms-key">"method"</span>: <span class="mms-str">"tools/call"</span>,
<span class="mms-key">"params"</span>: {
  <span class="mms-key">"name"</span>: <span class="mms-str">"search"</span>,
  <span class="mms-key">"arguments"</span>: { ... }
}</pre>
        <div class="mms-card-foot">ждёт ответ с тем же <code>id</code></div>
      </div>

      <!-- Response -->
      <div class="mms-card mms-card--resp" :class="{ 'mms-visible': showResp }">
        <div class="mms-card-head">
          <div class="mms-card-name">Response</div>
          <div class="mms-card-dir">сервер → клиент</div>
        </div>
        <pre class="mms-code"><span class="mms-key">"id"</span>: 1,
<span class="mms-key">"result"</span>: {
  <span class="mms-key">"content"</span>: [{
    <span class="mms-key">"type"</span>: <span class="mms-str">"text"</span>,
    <span class="mms-key">"text"</span>: <span class="mms-str">"..."</span>
  }]
}</pre>
        <div class="mms-card-foot">привязан к <code>id</code> запроса</div>
      </div>

      <!-- Notification -->
      <div class="mms-card mms-card--notify" :class="{ 'mms-visible': showNotify }">
        <div class="mms-card-head">
          <div class="mms-card-name">Notification</div>
          <div class="mms-card-dir">любая сторона</div>
        </div>
        <pre class="mms-code"><span class="mms-comment">// без id</span>
<span class="mms-key">"method"</span>: <span class="mms-str">"notifications/
  tools/list_changed"</span></pre>
        <div class="mms-card-foot">fire-and-forget · без ответа</div>
      </div>

    </div>

    <div class="mms-foot">
      только JSON · читаемость важнее производительности
    </div>

  </div>
</template>

<style scoped>
.mms-root {
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

.mms-title {
  font-size: 1.1em;
  font-weight: 700;
  color: #93c5fd;
  letter-spacing: 0.02em;
  align-self: flex-start;
}

.mms-title-sub {
  color: #64748b;
  font-weight: 400;
  font-size: 0.82em;
  font-family: 'JetBrains Mono', monospace;
}

/* Grid */
.mms-grid {
  flex: 1;
  display: flex;
  align-items: stretch;
  justify-content: center;
  gap: 16px;
  width: 100%;
}

.mms-card {
  flex: 1;
  display: flex;
  flex-direction: column;
  border-radius: 10px;
  padding: 12px 14px 10px;
  max-width: 290px;
  opacity: 0;
  transform: translateY(10px);
}

.mms-card.mms-visible {
  animation: mms-appear 0.35s ease-out both;
}

@keyframes mms-appear {
  from { opacity: 0; transform: translateY(10px); }
  to   { opacity: 1; transform: translateY(0); }
}

.mms-card--req {
  background: rgba(37, 99, 235, 0.14);
  border: 1.5px solid #3b82f6;
}

.mms-card--resp {
  background: rgba(16, 185, 129, 0.12);
  border: 1.5px solid #10b981;
}

.mms-card--notify {
  background: rgba(168, 85, 247, 0.12);
  border: 1.5px solid #a855f7;
}

/* Card header */
.mms-card-head {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  margin-bottom: 8px;
  gap: 8px;
}

.mms-card-name {
  font-weight: 700;
  font-size: 0.95em;
  color: #e2e8f0;
}

.mms-card-dir {
  font-size: 0.66em;
  color: #94a3b8;
  letter-spacing: 0.04em;
  font-family: 'JetBrains Mono', monospace;
}

/* Code block */
.mms-code {
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.66em;
  color: #cbd5e1;
  background: rgba(15, 23, 42, 0.55);
  border-radius: 6px;
  padding: 10px 12px;
  margin: 0;
  line-height: 1.55;
  white-space: pre;
  overflow: hidden;
  flex: 1;
}

.mms-key { color: #60a5fa; }
.mms-str { color: #fbbf24; }
.mms-comment { color: #64748b; font-style: italic; }

/* Card footer */
.mms-card-foot {
  margin-top: 8px;
  font-size: 0.68em;
  color: #94a3b8;
  text-align: center;
  font-style: italic;
}

.mms-card-foot code {
  font-family: 'JetBrains Mono', monospace;
  background: rgba(15, 23, 42, 0.5);
  padding: 1px 5px;
  border-radius: 3px;
  font-size: 0.92em;
  font-style: normal;
}

.mms-foot {
  font-size: 0.7em;
  color: #475569;
  letter-spacing: 0.06em;
  font-style: italic;
}
</style>
