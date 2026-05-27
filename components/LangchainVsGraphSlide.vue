<script setup>
import { useNav } from '@slidev/client'
import { computed } from 'vue'

const { clicks } = useNav()

const showChain = computed(() => clicks.value >= 1)
const showGraph = computed(() => clicks.value >= 2)
</script>

<template>
  <div class="lvg-root">

    <div class="lvg-title">LangChain vs LangGraph — главное отличие</div>

    <div class="lvg-grid">

      <!-- LangChain side -->
      <div class="lvg-col lvg-col--chain" :class="{ 'lvg-on': showChain }">
        <div class="lvg-col-head">
          <div class="lvg-col-name">LangChain</div>
          <div class="lvg-col-sub">линейная цепочка</div>
        </div>

        <!-- Linear flow A → B → C -->
        <div class="lvg-chain-flow">
          <div class="lvg-node">A</div>
          <div class="lvg-arrow">→</div>
          <div class="lvg-node">B</div>
          <div class="lvg-arrow">→</div>
          <div class="lvg-node">C</div>
        </div>

        <div class="lvg-use">
          <div class="lvg-use-label">Когда брать</div>
          <ul class="lvg-use-list">
            <li>RAG-пайплайн</li>
            <li>один агент с инструментами</li>
            <li>быстрый прототип</li>
          </ul>
        </div>
      </div>

      <!-- LangGraph side -->
      <div class="lvg-col lvg-col--graph" :class="{ 'lvg-on': showGraph }">
        <div class="lvg-col-head">
          <div class="lvg-col-name">LangGraph</div>
          <div class="lvg-col-sub">граф состояний (FSM)</div>
        </div>

        <!-- Graph with cycles and branches -->
        <svg class="lvg-graph-svg" viewBox="0 0 280 130">
          <defs>
            <marker id="lvg-arr" markerWidth="6" markerHeight="6" refX="5" refY="3" orient="auto">
              <polygon points="0,0.5 0,5.5 6,3" fill="#a855f7"/>
            </marker>
          </defs>

          <!-- Edges -->
          <path d="M50,65 L110,65" stroke="#a855f7" stroke-width="1.6" fill="none" marker-end="url(#lvg-arr)"/>
          <!-- branch up -->
          <path d="M140,52 C160,40 180,30 215,30" stroke="#a855f7" stroke-width="1.6" fill="none" marker-end="url(#lvg-arr)"/>
          <!-- branch down -->
          <path d="M140,78 C160,90 180,100 215,100" stroke="#a855f7" stroke-width="1.6" fill="none" marker-end="url(#lvg-arr)"/>
          <!-- cycle back -->
          <path d="M215,30 C260,30 260,100 230,100" stroke="#10b981" stroke-width="1.5" fill="none" marker-end="url(#lvg-arr)" stroke-dasharray="4 3"/>

          <!-- Nodes -->
          <g>
            <circle cx="35" cy="65" r="15" fill="rgba(168,85,247,0.2)" stroke="#a855f7" stroke-width="1.5"/>
            <text x="35" y="69" font-family="JetBrains Mono" font-size="11" fill="#c4b5fd" text-anchor="middle">A</text>
          </g>
          <g>
            <circle cx="125" cy="65" r="15" fill="rgba(168,85,247,0.2)" stroke="#a855f7" stroke-width="1.5"/>
            <text x="125" y="69" font-family="JetBrains Mono" font-size="11" fill="#c4b5fd" text-anchor="middle">B</text>
          </g>
          <g>
            <circle cx="230" cy="30" r="15" fill="rgba(168,85,247,0.2)" stroke="#a855f7" stroke-width="1.5"/>
            <text x="230" y="34" font-family="JetBrains Mono" font-size="11" fill="#c4b5fd" text-anchor="middle">C</text>
          </g>
          <g>
            <circle cx="230" cy="100" r="15" fill="rgba(168,85,247,0.2)" stroke="#a855f7" stroke-width="1.5"/>
            <text x="230" y="104" font-family="JetBrains Mono" font-size="11" fill="#c4b5fd" text-anchor="middle">D</text>
          </g>
        </svg>

        <div class="lvg-features">
          <span class="lvg-feat">↺ циклы</span>
          <span class="lvg-feat">⤴ ветвления</span>
          <span class="lvg-feat">👤 human-in-the-loop</span>
          <span class="lvg-feat">💾 persistent state</span>
        </div>

        <div class="lvg-use">
          <div class="lvg-use-label">Когда брать</div>
          <ul class="lvg-use-list">
            <li>серьёзная агентная система</li>
            <li>сложное состояние</li>
            <li>откаты и долгие сессии</li>
          </ul>
        </div>
      </div>

    </div>

    <div class="lvg-foot">LangGraph построен поверх LangChain — это два слоя одной экосистемы</div>

  </div>
</template>

<style scoped>
.lvg-root {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 22px 28px 14px;
  box-sizing: border-box;
  gap: 14px;
}

.lvg-title {
  font-size: 1.1em;
  font-weight: 700;
  color: #93c5fd;
  letter-spacing: 0.02em;
  align-self: flex-start;
}

.lvg-grid {
  flex: 1;
  display: flex;
  align-items: stretch;
  justify-content: center;
  gap: 22px;
  width: 100%;
}

.lvg-col {
  flex: 1;
  max-width: 380px;
  border-radius: 12px;
  padding: 14px 18px 12px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  opacity: 0;
  transform: translateY(8px);
  transition: opacity 0.4s, transform 0.4s;
}

.lvg-col.lvg-on {
  opacity: 1;
  transform: translateY(0);
}

.lvg-col--chain {
  background: rgba(37, 99, 235, 0.12);
  border: 1.5px solid #3b82f6;
}

.lvg-col--graph {
  background: rgba(168, 85, 247, 0.10);
  border: 1.5px solid #a855f7;
}

.lvg-col-head {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
}

.lvg-col-name {
  font-size: 1.1em;
  font-weight: 700;
  color: #e2e8f0;
}

.lvg-col-sub {
  font-size: 0.7em;
  color: #94a3b8;
  font-style: italic;
  letter-spacing: 0.04em;
}

/* Chain flow */
.lvg-chain-flow {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 16px 0;
}

.lvg-node {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  background: rgba(37, 99, 235, 0.22);
  border: 1.5px solid #3b82f6;
  color: #93c5fd;
  font-family: 'JetBrains Mono', monospace;
  font-weight: 700;
  font-size: 0.9em;
  display: flex;
  align-items: center;
  justify-content: center;
}

.lvg-arrow {
  font-size: 1.1em;
  color: #3b82f6;
  font-weight: 700;
}

/* Graph SVG */
.lvg-graph-svg {
  width: 280px;
  height: 130px;
}

/* Features chips */
.lvg-features {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 6px;
}

.lvg-feat {
  background: rgba(168, 85, 247, 0.15);
  border: 1px solid rgba(168, 85, 247, 0.4);
  color: #c4b5fd;
  border-radius: 999px;
  padding: 2px 10px;
  font-size: 0.66em;
  font-weight: 500;
}

/* Use list */
.lvg-use {
  width: 100%;
  margin-top: auto;
  padding-top: 8px;
  border-top: 1px solid rgba(100, 116, 139, 0.2);
}

.lvg-use-label {
  font-size: 0.66em;
  color: #64748b;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  margin-bottom: 4px;
}

.lvg-use-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.lvg-use-list li {
  font-size: 0.72em;
  color: #cbd5e1;
  padding: 2px 0 2px 14px;
  position: relative;
}

.lvg-use-list li::before {
  content: '·';
  position: absolute;
  left: 4px;
  color: #475569;
  font-weight: 700;
}

.lvg-foot {
  font-size: 0.7em;
  color: #475569;
  font-style: italic;
  text-align: center;
}
</style>
