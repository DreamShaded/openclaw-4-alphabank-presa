<script setup>
import { useNav } from '@slidev/client'
import { computed } from 'vue'

const { clicks } = useNav()

// Each click reveals one more connection (plain ComputedRefs — auto-unwrap in templates)
const s1 = computed(() => clicks.value >= 1) // Браузер
const s2 = computed(() => clicks.value >= 2) // Мессенджер
const s3 = computed(() => clicks.value >= 3) // Почта
const s4 = computed(() => clicks.value >= 4) // Интернет
const s5 = computed(() => clicks.value >= 5) // Операционная система
</script>

<template>
  <div class="cc-root">
    <!-- OpenClaw: fixed on left-center (same style as moved state in slide 1) -->
    <div class="cc-oc">
      <div class="cc-oc-box">OpenClaw</div>
    </div>

    <!-- Target blocks: appear one by one per click -->
    <div class="cc-targets">
      <div class="cc-target" :class="{ 'cc-visible': s1 }">Браузер</div>
      <div class="cc-target" :class="{ 'cc-visible': s2 }">Мессенджер</div>
      <div class="cc-target" :class="{ 'cc-visible': s3 }">Почта</div>
      <div class="cc-target" :class="{ 'cc-visible': s4 }">Интернет</div>
      <div class="cc-target" :class="{ 'cc-visible': s5 }">Операционная система</div>
    </div>

    <!--
      SVG arrows overlay — viewBox=980×552 (Slidev default canvas).
      Start X=249: left(4%×980=39) + width(210px) = 249. Y=276 (vertical center).
      End X=539: left(55%×980). Target Y = block centers, blockH≈45px, gap=14px:
        Браузер      → Y=158
        Мессенджер   → Y=217
        Почта        → Y=276  (center — straight horizontal)
        Интернет     → Y=335
        Операционная → Y=394
    -->
    <svg class="cc-svg" viewBox="0 0 980 552" xmlns="http://www.w3.org/2000/svg">
      <defs>
        <marker id="cc-arr" markerWidth="7" markerHeight="7" refX="6" refY="3.5" orient="auto">
          <polygon points="0,0.5 0,6.5 7,3.5" fill="#3b82f6"/>
        </marker>
      </defs>

      <!-- Arrow 1: Браузер (curves up) -->
      <path v-if="s1"
        class="cc-path"
        d="M249,276 C380,276 380,158 539,158"
        pathLength="1"
        marker-end="url(#cc-arr)"/>

      <!-- Arrow 2: Мессенджер (slight curve up) -->
      <path v-if="s2"
        class="cc-path"
        d="M249,276 C380,276 380,217 539,217"
        pathLength="1"
        marker-end="url(#cc-arr)"/>

      <!-- Arrow 3: Почта (straight horizontal — OpenClaw and Почта share center Y=276) -->
      <path v-if="s3"
        class="cc-path"
        d="M249,276 L539,276"
        pathLength="1"
        marker-end="url(#cc-arr)"/>

      <!-- Arrow 4: Интернет (slight curve down) -->
      <path v-if="s4"
        class="cc-path"
        d="M249,276 C380,276 380,335 539,335"
        pathLength="1"
        marker-end="url(#cc-arr)"/>

      <!-- Arrow 5: Операционная система (curves down) -->
      <path v-if="s5"
        class="cc-path"
        d="M249,276 C380,276 380,394 539,394"
        pathLength="1"
        marker-end="url(#cc-arr)"/>
    </svg>
  </div>
</template>

<style scoped>
/* ── Root ───────────────────────────────────────── */
.cc-root {
  position: relative;
  width: 100%;
  height: 100%;
}

/* ── OpenClaw box (left side) ───────────────────── */
.cc-oc {
  position: absolute;
  left: 4%;
  top: 50%;
  transform: translateY(-50%);
  width: 210px;
}

.cc-oc-box {
  background: rgba(37, 99, 235, 0.18);
  border: 2px solid #3b82f6;
  color: #93c5fd;
  font-size: 1.35em;
  font-weight: 700;
  padding: 14px 22px;
  text-align: center;
  border-radius: 10px;
  box-shadow: 0 0 18px rgba(59, 130, 246, 0.22);
  letter-spacing: 0.04em;
  user-select: none;
}

/* ── Target column ──────────────────────────────── */
/* Positioned at ~55% from left; centered vertically.
   5 blocks × ~52px + 4 gaps × 14px ≈ 316px total → top ~118px */
.cc-targets {
  position: absolute;
  left: 55%;
  top: 50%;
  transform: translateY(-50%);
  display: flex;
  flex-direction: column;
  gap: 14px;
  width: 33%;
}

.cc-target {
  background: rgba(71, 85, 105, 0.12);
  border: 1.5px solid #475569;
  color: #94a3b8;
  border-radius: 8px;
  padding: 12px 18px;
  text-align: center;
  font-weight: 600;
  font-size: 0.95em;
  opacity: 0;
  user-select: none;
}

/* Reveal animation: slide in from right */
.cc-target.cc-visible {
  animation: cc-appear 0.32s ease-out both;
}

@keyframes cc-appear {
  from { opacity: 0; transform: translateX(16px); }
  to   { opacity: 1; transform: translateX(0); }
}

/* ── SVG overlay ────────────────────────────────── */
.cc-svg {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  overflow: visible;
}

/* Arrow draw animation using pathLength="1" trick */
.cc-path {
  fill: none;
  stroke: #3b82f6;
  stroke-width: 1.8;
  stroke-linecap: round;
  stroke-dasharray: 1;
  stroke-dashoffset: 1;
  animation: cc-draw 0.48s ease-out both;
}

@keyframes cc-draw {
  to { stroke-dashoffset: 0; }
}
</style>
