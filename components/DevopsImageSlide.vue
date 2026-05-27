<script setup>
/**
 * DevopsImageSlide — full-slide centered image.
 * Imports via Vite so the asset is processed correctly (avoids Slidev markdown
 * src-as-module-import path resolution bug on Windows with non-ASCII filenames).
 */
import devopsImg from '../public/images/devops.jpg'
</script>

<template>
  <div class="dis-root">
    <img :src="devopsImg" class="dis-img" alt="DevOps" />
  </div>
</template>

<style scoped>
.dis-root {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 24px;
  box-sizing: border-box;
}

.dis-img {
  max-height: 100%;
  max-width: 100%;
  object-fit: contain;
  border-radius: 12px;
}
</style>
