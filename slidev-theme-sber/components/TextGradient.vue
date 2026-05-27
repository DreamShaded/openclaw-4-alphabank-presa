<script setup lang="ts">
import { computed } from "vue";

const props = defineProps<{
	tag?: string;
}>();

const Tag = computed(() => props.tag || "span");
</script>

<template>
  <component :is="Tag" class="text-gradient">
    <slot />
  </component>
</template>

<style scoped>
.text-gradient {
  background: var(--color-accent-gradient);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  font-weight: bold;
}
</style>

