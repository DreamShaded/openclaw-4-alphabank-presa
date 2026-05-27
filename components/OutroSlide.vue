<script setup>
import qrImg from '../public/images/qr-github.png'
</script>

<template>
  <div class="out-root">

    <div class="out-left">
      <div class="out-thanks">Спасибо</div>
      <div class="out-tag">за внимание</div>

      <div class="out-info">
        <div class="out-info-line">
          <span class="out-info-label">Доклад</span>
          <span class="out-info-text">OpenClaw, скиллы и корпоративная рутина</span>
        </div>
        <div class="out-info-line">
          <span class="out-info-label">Автор</span>
          <span class="out-info-text">Троицкий Роман · ПАО Сбербанк · SmartCare</span>
        </div>
      </div>

      <div class="out-cta">
        <span>код демо и скиллов — в репозитории</span>
      </div>
    </div>

    <div class="out-right">
      <div class="out-qr-card">
        <img :src="qrImg" alt="QR · dreamshaded.github.io/openclaw-4-alphabank-presa" class="out-qr-img" />
        <div class="out-qr-cap">наведите камеру</div>
      </div>
      <div class="out-url">
        <span class="out-url-domain">dreamshaded.github.io/</span>openclaw-4-alphabank-presa
      </div>
    </div>

  </div>
</template>

<style scoped>
.out-root {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 56px;
  padding: 40px 56px;
  box-sizing: border-box;
}

/* Left column */
.out-left {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 22px;
  max-width: 500px;
}

.out-thanks {
  font-size: 3.4em;
  font-weight: 800;
  color: #93c5fd;
  line-height: 1;
  letter-spacing: -0.02em;
  text-shadow: 0 0 30px rgba(59, 130, 246, 0.25);
}

.out-tag {
  font-size: 1.4em;
  color: #64748b;
  font-style: italic;
  margin-top: -8px;
}

.out-info {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-top: 12px;
}

.out-info-line {
  display: flex;
  align-items: baseline;
  gap: 12px;
  font-size: 0.85em;
}

.out-info-label {
  font-size: 0.75em;
  color: #475569;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  min-width: 70px;
}

.out-info-text {
  color: #cbd5e1;
}

.out-cta {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-top: 8px;
  padding: 8px 16px;
  background: rgba(37, 99, 235, 0.10);
  border: 1px dashed rgba(59, 130, 246, 0.4);
  border-radius: 999px;
  font-size: 0.85em;
  color: #93c5fd;
}

.out-cta-arrow {
  font-weight: 700;
  font-size: 1.2em;
}

/* Right column */
.out-right {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
}

.out-qr-card {
  background: #ffffff;
  padding: 16px;
  border-radius: 16px;
  box-shadow: 0 0 38px rgba(59, 130, 246, 0.35),
              0 8px 24px rgba(0, 0, 0, 0.4);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  border: 2px solid #3b82f6;
}

.out-qr-img {
  width: 260px;
  height: 260px;
  display: block;
}

.out-qr-cap {
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.7em;
  color: #475569;
  letter-spacing: 0.04em;
}

.out-url {
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.72em;
  color: #cbd5e1;
  letter-spacing: 0.01em;
  background: rgba(15, 23, 42, 0.6);
  border: 1px solid #334155;
  border-radius: 6px;
  padding: 6px 14px;
}

.out-url-domain {
  color: #60a5fa;
}
</style>
