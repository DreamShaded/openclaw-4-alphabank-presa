<script setup>
import { useNav } from '@slidev/client'
import { computed } from 'vue'

const { clicks } = useNav()

// OpenClaw moves left on first click
const moved = computed(() => clicks.value >= 1)
// Right-side ecosystem appears on first click
const show = computed(() => clicks.value >= 1)
</script>

<template>
  <div class="eco-root">
    <!-- OpenClaw: top-center initially → left-center on click -->
    <div class="oc-wrap" :class="{ 'oc-moved': moved }">
      <div class="oc-box oc-main">OpenClaw</div>
    </div>

    <!-- Right ecosystem: IronClaw, PicoClaw, KimiClaw, AnyOtherClaw -->
    <div v-if="show" class="eco-right">
      <div class="oc-box oc-peer eco-a1">IronClaw</div>
      <div class="oc-box oc-peer eco-a2">PicoClaw</div>
      <div class="oc-box oc-peer eco-a3">KimiClaw</div>
      <div class="oc-box oc-dim eco-a4">AnyOtherClaw</div>
    </div>
  </div>
</template>

<style scoped>
/* ── Root container ─────────────────────────────── */
.eco-root {
  position: relative;
  width: 100%;
  height: 100%;
}

/* ── OpenClaw wrapper: handles position transition ─ */
.oc-wrap {
  position: absolute;
  /* Initial: centered horizontally, near top.
     Use translate(x,y) — not translateX — so CSS can interpolate smoothly to moved state. */
  left: 50%;
  top: 13%;
  transform: translate(-50%, 0);
  width: 280px;
  transition: left 0.65s cubic-bezier(0.4, 0, 0.2, 1),
              top  0.65s cubic-bezier(0.4, 0, 0.2, 1),
              transform 0.65s cubic-bezier(0.4, 0, 0.2, 1),
              width 0.65s cubic-bezier(0.4, 0, 0.2, 1);
  z-index: 5;
}

/* Moved state: left side, vertically centered.
   translate(0,-50%) matches the two-arg form used in initial state — smooth interpolation. */
.oc-wrap.oc-moved {
  left: 4%;
  top: 50%;
  transform: translate(0, -50%);
  width: 210px;
}

/* ── Box base ───────────────────────────────────── */
.oc-box {
  border-radius: 10px;
  padding: 14px 22px;
  text-align: center;
  font-weight: 700;
  letter-spacing: 0.04em;
  user-select: none;
}

/* OpenClaw: blue glow */
.oc-main {
  background: rgba(37, 99, 235, 0.18);
  border: 2px solid #3b82f6;
  color: #93c5fd;
  font-size: 1.4em;
  box-shadow: 0 0 18px rgba(59, 130, 246, 0.22);
}

/* Secondary claws: subtle slate */
.oc-peer {
  background: rgba(100, 116, 139, 0.15);
  border: 1.5px solid #64748b;
  color: #cbd5e1;
  font-size: 1.05em;
}

/* AnyOtherClaw: dimmer, dashed, smaller */
.oc-dim {
  background: rgba(71, 85, 105, 0.08);
  border: 1px dashed #475569;
  color: #94a3b8;
  font-size: 0.88em;
}

/* ── Right ecosystem column ─────────────────────── */
.eco-right {
  position: absolute;
  right: 4%;
  top: 50%;
  transform: translateY(-50%);
  width: 44%;
  display: flex;
  flex-direction: column;
  gap: 13px;
}

/* ── Staggered slide-in animation ───────────────── */
@keyframes eco-in {
  from { opacity: 0; transform: translateX(26px); }
  to   { opacity: 1; transform: translateX(0); }
}

.eco-a1 { animation: eco-in 0.35s ease-out 0.05s both; }
.eco-a2 { animation: eco-in 0.35s ease-out 0.18s both; }
.eco-a3 { animation: eco-in 0.35s ease-out 0.31s both; }
.eco-a4 { animation: eco-in 0.35s ease-out 0.44s both; }
</style>
