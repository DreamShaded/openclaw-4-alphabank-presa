<script setup>
import { useNav } from '@slidev/client'
import { computed } from 'vue'

const { clicks } = useNav()

const showVec   = computed(() => clicks.value >= 1)
const showSpace = computed(() => clicks.value >= 2)
</script>

<template>
  <div class="em-root">

    <div class="em-title">Эмбеддинг</div>

    <div class="em-layout">

      <!-- Left: text → vector -->
      <div class="em-conversion">
        <div class="em-text-row">
          <div class="em-text">«кошка»</div>
          <div class="em-arrow">→</div>
          <div class="em-vec" :class="{ 'em-on': showVec }">
            [ 0.21, -0.84, 0.07, ..., 0.55 ]
          </div>
        </div>
        <div class="em-text-row">
          <div class="em-text">«котёнок»</div>
          <div class="em-arrow">→</div>
          <div class="em-vec" :class="{ 'em-on': showVec }">
            [ 0.18, -0.79, 0.12, ..., 0.51 ]
          </div>
        </div>
        <div class="em-text-row">
          <div class="em-text">«налоговый кодекс»</div>
          <div class="em-arrow">→</div>
          <div class="em-vec" :class="{ 'em-on': showVec }">
            [ -0.62, 0.34, 0.91, ..., -0.18 ]
          </div>
        </div>

        <div class="em-spec" :class="{ 'em-on': showVec }">
          ~1024 числа на любой текст · от слова до страницы
        </div>
      </div>

      <!-- Right: vector space (HTML for reliable text sizing) -->
      <div class="em-space" :class="{ 'em-on': showSpace }">
        <div class="em-space-label">пространство смыслов</div>

        <div class="em-canvas">
          <!-- кошка — top-left area -->
          <div class="em-point em-point--blue" style="left: 22%; top: 28%;">
            <span class="em-dot"></span>
            <span class="em-pt-label em-pt-label--right">кошка</span>
          </div>
          <!-- котёнок — close to кошка -->
          <div class="em-point em-point--blue" style="left: 38%; top: 20%;">
            <span class="em-dot"></span>
            <span class="em-pt-label em-pt-label--right">котёнок</span>
          </div>
          <!-- налоговый кодекс — far bottom-right -->
          <div class="em-point em-point--purple" style="left: 70%; top: 72%;">
            <span class="em-dot em-dot--purple"></span>
            <span class="em-pt-label em-pt-label--purple em-pt-label--below">налоговый кодекс</span>
          </div>
        </div>
      </div>

    </div>

  </div>
</template>

<style scoped>
.em-root {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 22px 32px 14px;
  box-sizing: border-box;
  gap: 14px;
}

.em-title {
  font-size: 1.1em;
  font-weight: 700;
  color: #93c5fd;
  letter-spacing: 0.02em;
  align-self: flex-start;
}

.em-layout {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 28px;
  width: 100%;
}

/* Left: text → vector */
.em-conversion {
  display: flex;
  flex-direction: column;
  gap: 10px;
  max-width: 460px;
}

.em-text-row {
  display: flex;
  align-items: center;
  gap: 10px;
}

.em-text {
  background: rgba(15, 23, 42, 0.55);
  border: 1px solid #334155;
  border-radius: 6px;
  padding: 6px 14px;
  font-size: 0.78em;
  color: #cbd5e1;
  font-style: italic;
  min-width: 160px;
  text-align: center;
}

.em-arrow {
  font-size: 1em;
  color: #475569;
}

.em-vec {
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.66em;
  color: #64748b;
  background: rgba(15, 23, 42, 0.4);
  border: 1px dashed #334155;
  border-radius: 6px;
  padding: 6px 10px;
  flex: 1;
  opacity: 0.3;
  transition: opacity 0.35s, color 0.35s, border-color 0.35s;
}

.em-vec.em-on {
  opacity: 1;
  color: #93c5fd;
  border-color: #3b82f6;
}

.em-spec {
  font-size: 0.7em;
  color: #475569;
  font-style: italic;
  text-align: center;
  margin-top: 6px;
  opacity: 0;
  transition: opacity 0.4s;
}

.em-spec.em-on { opacity: 1; }

/* Right: vector space */
.em-space {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  padding: 14px 18px 10px;
  border: 1.5px solid #3b82f6;
  border-radius: 12px;
  background: rgba(37, 99, 235, 0.06);
  opacity: 0;
  transform: scale(0.95);
  transition: opacity 0.35s, transform 0.35s;
}

.em-space.em-on {
  opacity: 1;
  transform: scale(1);
}

.em-space-label {
  font-size: 0.7em;
  color: #60a5fa;
  text-transform: uppercase;
  letter-spacing: 0.08em;
}

/* Canvas: relative-positioned area for dots/labels */
.em-canvas {
  position: relative;
  width: 280px;
  height: 200px;
  border-left: 1px dashed #334155;
  border-bottom: 1px dashed #334155;
  margin: 4px 8px 4px 16px;
}

.em-point {
  position: absolute;
  display: flex;
  align-items: center;
  gap: 6px;
  transform: translate(-50%, -50%);
  white-space: nowrap;
}

.em-dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: #3b82f6;
  box-shadow: 0 0 8px rgba(59, 130, 246, 0.5);
}

.em-dot--purple {
  background: #a855f7;
  box-shadow: 0 0 8px rgba(168, 85, 247, 0.5);
}

.em-pt-label {
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.72em;
  color: #93c5fd;
  font-weight: 500;
}

.em-pt-label--purple { color: #c4b5fd; }

.em-pt-label--right { order: 1; }
.em-pt-label--below {
  position: absolute;
  top: 16px;
  left: 50%;
  transform: translateX(-50%);
}

.em-foot {
  font-size: 0.74em;
  color: #475569;
  display: flex;
  align-items: center;
  gap: 12px;
}

.em-pro { color: #6ee7b7; }
.em-con { color: #fca5a5; }
.em-sep { color: #334155; }
</style>
