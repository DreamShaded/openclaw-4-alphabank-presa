<script setup lang="ts">
import { computed } from "vue";
import TextGradient from "./TextGradient.vue";

const props = defineProps<{
	tag?: string;
}>();

const Tag = computed(() => props.tag || "p");
</script>

<template>
  <component :is="Tag" class="text-big">
    <TextGradient>
      <slot />
    </TextGradient>
  </component>
</template>

<style scoped>
.text-big {
  font-size: 5rem; /* ~80px при базовом размере 16px */
  line-height: 1.1;
  font-weight: bold;
  text-transform: uppercase;
}
</style>

