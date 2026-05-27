<script setup>
import { useNav } from '@slidev/client'
import { computed } from 'vue'

const { clicks } = useNav()

const showGrid = computed(() => clicks.value >= 1)
const showMain = computed(() => clicks.value >= 2)

const skills = [
  { icon: '📧', name: 'Почта',         desc: 'читает, фильтрует, саммари' },
  { icon: '🌐', name: 'Сайты',         desc: 'мониторинг нужных источников' },
  { icon: '💪', name: 'Тренировки',    desc: 'прогресс и нагрузка' },
  { icon: '😴', name: 'Самочувствие',  desc: 'HRV, сон, усталость' },
  { icon: '🏖️', name: 'Отпуска',       desc: 'кто и когда отсутствует' },
  { icon: '🔧', name: 'Ремонт',        desc: 'напоминалки по дому' },
  { icon: '🍳', name: 'Рецепты',       desc: 'из инстаграма в заметки' },
  { icon: '🛒', name: 'Быт',           desc: 'задачи и закупки' },
]
</script>

<template>
  <div class="wu-root">

    <div class="wu-header">
      <div class="wu-title">Что я использую</div>
      <div class="wu-badge">пара десятков скиллов · ежедневно</div>
    </div>

    <div class="wu-layout">

      <!-- Left: daily skill cards -->
      <div class="wu-left">
        <div
          v-for="(s, i) in skills"
          :key="i"
          class="wu-tag"
          :class="{ 'wu-on': showGrid }"
          :style="showGrid ? `animation-delay: ${i * 0.07}s` : ''"
        >
          <span class="wu-icon">{{ s.icon }}</span>
          <div class="wu-body">
            <div class="wu-name">{{ s.name }}</div>
            <div class="wu-desc">{{ s.desc }}</div>
          </div>
        </div>
      </div>

      <!-- Right: main planning skill -->
      <div class="wu-right" :class="{ 'wu-on': showMain }">
        <div class="wu-glow" />
        <div class="wu-inner">
          <div class="wu-main-icon">📅</div>
          <div class="wu-main-name">Планирование</div>
          <div class="wu-main-sub">с учётом календаря · по подзадачам</div>
          <div class="wu-divider" />
          <div class="wu-nodes">
            <div class="wu-node">📆 &nbsp;Google Calendar</div>
            <div class="wu-node">🏠 &nbsp;Инфраструктура дома</div>
            <div class="wu-node">🔑 &nbsp;Доступы</div>
          </div>
          <div class="wu-quote">«отдаю почти как есть»</div>
        </div>
      </div>

    </div>
  </div>
</template>

<style scoped>
.wu-root {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  padding: 28px 48px 20px;
  box-sizing: border-box;
  gap: 14px;
}

/* Header */
.wu-header {
  display: flex;
  align-items: baseline;
  gap: 16px;
}

.wu-title {
  font-size: 1.35em;
  font-weight: 700;
  color: #93c5fd;
  letter-spacing: 0.02em;
}

.wu-badge {
  font-size: 0.74em;
  color: #475569;
  letter-spacing: 0.04em;
}

/* Two-column layout */
.wu-layout {
  flex: 1;
  display: grid;
  grid-template-columns: 1fr 310px;
  gap: 24px;
  min-height: 0;
}

/* Left: 2×4 grid of skill tags */
.wu-left {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
  align-content: start;
}

.wu-tag {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 10px 14px;
  border-radius: 10px;
  background: rgba(71, 85, 105, 0.10);
  border: 1px solid #334155;
  opacity: 0;
  transform: translateX(-8px);
}

.wu-tag.wu-on {
  animation: wu-in 0.32s ease-out both;
}

@keyframes wu-in {
  from { opacity: 0; transform: translateX(-8px); }
  to   { opacity: 1; transform: translateX(0); }
}

.wu-icon {
  font-size: 1.45em;
  line-height: 1;
  flex-shrink: 0;
}

.wu-body {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.wu-name {
  font-size: 0.88em;
  font-weight: 600;
  color: #e2e8f0;
}

.wu-desc {
  font-size: 0.7em;
  color: #64748b;
  line-height: 1.3;
}

/* Right: main planning skill */
.wu-right {
  position: relative;
  border-radius: 16px;
  background: linear-gradient(145deg, rgba(37, 99, 235, 0.16), rgba(168, 85, 247, 0.12));
  border: 2px solid rgba(59, 130, 246, 0.65);
  box-shadow: 0 0 36px rgba(59, 130, 246, 0.18);
  overflow: hidden;
  opacity: 0;
  transform: translateY(10px) scale(0.97);
  transition: opacity 0.42s ease-out, transform 0.42s ease-out;
}

.wu-right.wu-on {
  opacity: 1;
  transform: translateY(0) scale(1);
}

.wu-glow {
  position: absolute;
  top: -50px;
  right: -50px;
  width: 200px;
  height: 200px;
  background: radial-gradient(circle, rgba(59, 130, 246, 0.22) 0%, transparent 70%);
  pointer-events: none;
}

.wu-inner {
  position: relative;
  z-index: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 22px 18px 18px;
  gap: 10px;
  height: 100%;
  box-sizing: border-box;
}

.wu-main-icon {
  font-size: 2em;
  line-height: 1;
}

.wu-main-name {
  font-size: 1.15em;
  font-weight: 700;
  color: #93c5fd;
  letter-spacing: 0.01em;
}

.wu-main-sub {
  font-size: 0.75em;
  color: #94a3b8;
  text-align: center;
  line-height: 1.5;
}

.wu-divider {
  width: 55%;
  height: 1px;
  background: rgba(59, 130, 246, 0.28);
  margin: 2px 0;
}

.wu-nodes {
  display: flex;
  flex-direction: column;
  gap: 8px;
  width: 100%;
}

.wu-node {
  padding: 7px 14px;
  background: rgba(59, 130, 246, 0.08);
  border: 1px solid rgba(59, 130, 246, 0.22);
  border-radius: 8px;
  font-size: 0.78em;
  color: #93c5fd;
  text-align: center;
}

.wu-quote {
  margin-top: auto;
  font-size: 0.72em;
  color: #475569;
  font-style: italic;
}
</style>
