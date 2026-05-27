<script setup lang="ts">
withDefaults(defineProps<{
  /** 0 = nothing highlighted, 1 = polyrepo, 2 = monorepo */
  highlight?: number;
}>(), {
  highlight: 0,
});
</script>

<template>
  <div class="scale-wrapper">
    <div class="scale">
      <!-- Endpoints -->
      <div class="endpoint left">
        <div class="endpoint-diamond" />
        <span class="endpoint-label">Изоляция</span>
      </div>

      <div class="endpoint right">
        <div class="endpoint-diamond" />
        <span class="endpoint-label">Интеграция</span>
      </div>

      <!-- Line -->
      <div class="line" />

      <!-- Points -->
      <div
        class="point polyrepo"
        :class="{ active: highlight === 1, dimmed: highlight === 2 }"
      >
        <div class="point-dot" />
        <div class="point-label">Полирепо</div>
        <div class="point-glow" />
      </div>

      <div
        class="point monorepo"
        :class="{ active: highlight === 2, dimmed: highlight === 1 }"
      >
        <div class="point-dot" />
        <div class="point-label">Монорепо</div>
        <div class="point-glow" />
      </div>
    </div>
  </div>
</template>

<style scoped>
.scale-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
  padding: 2rem;
}

.scale {
  position: relative;
  width: 100%;
  max-width: 800px;
  height: 200px;
}

/* Main line */
.line {
  position: absolute;
  top: 50%;
  left: 40px;
  right: 40px;
  height: 2px;
  background: linear-gradient(
    90deg,
    rgba(255, 255, 255, 0.15) 0%,
    rgba(255, 255, 255, 0.4) 50%,
    rgba(255, 255, 255, 0.15) 100%
  );
  transform: translateY(-50%);
}

/* Endpoints */
.endpoint {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  z-index: 2;
}

.endpoint.left {
  left: 0;
}

.endpoint.right {
  right: 0;
}

.endpoint-diamond {
  width: 12px;
  height: 12px;
  background: rgba(255, 255, 255, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.5);
  transform: rotate(45deg);
}

.endpoint-label {
  position: absolute;
  top: calc(100% + 0.75rem);
  white-space: nowrap;
  font-size: 0.95rem;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.5);
  letter-spacing: 0.05em;
  text-transform: uppercase;
  font-family: 'SB Sans Text', sans-serif;
}

/* Points */
.point {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  z-index: 3;
  transition: all 0.5s cubic-bezier(0.4, 0, 0.2, 1);
}

.point.polyrepo {
  left: 30%;
}

.point.monorepo {
  left: 65%;
}

.point-dot {
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.25);
  border: 2px solid rgba(255, 255, 255, 0.4);
  transition: all 0.5s cubic-bezier(0.4, 0, 0.2, 1);
  position: relative;
  z-index: 2;
}

.point-label {
  position: absolute;
  bottom: calc(100% + 1rem);
  white-space: nowrap;
  font-size: 1.2rem;
  font-weight: 700;
  color: rgba(255, 255, 255, 0.6);
  font-family: 'SB Sans Display', sans-serif;
  transition: all 0.5s cubic-bezier(0.4, 0, 0.2, 1);
}

.point-glow {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 60px;
  height: 60px;
  border-radius: 50%;
  transform: translate(-50%, -50%);
  opacity: 0;
  transition: all 0.5s cubic-bezier(0.4, 0, 0.2, 1);
  z-index: 1;
}

/* Active state */
.point.active .point-dot {
  width: 22px;
  height: 22px;
  background: #56ff71;
  border-color: #56ff71;
  box-shadow:
    0 0 20px rgba(86, 255, 113, 0.4),
    0 0 40px rgba(86, 255, 113, 0.15);
}

.point.active .point-label {
  color: #fff;
  transform: scale(1.1);
  text-shadow: 0 0 20px rgba(86, 255, 113, 0.3);
}

.point.active .point-glow {
  opacity: 1;
  background: radial-gradient(circle, rgba(86, 255, 113, 0.2) 0%, transparent 70%);
  width: 100px;
  height: 100px;
}

/* Dimmed state */
.point.dimmed .point-dot {
  opacity: 0.3;
}

.point.dimmed .point-label {
  opacity: 0.3;
}
</style>
