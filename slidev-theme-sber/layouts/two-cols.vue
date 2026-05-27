<script setup>
import { useNav } from "@slidev/client";
import { computed } from "vue";
import { isDark } from "@slidev/client/logic/dark.ts";

// Background logic copied from simple-slide.vue
const backgroundsDark = import.meta.glob(
	"../assets/images/backgrounds/dark/regular-slides/*.jpg",
	{ eager: true, query: "?url", import: "default" },
);
const backgroundsLight = import.meta.glob(
	"../assets/images/backgrounds/light/regular-slides/*.jpg",
	{ eager: true, query: "?url", import: "default" },
);

const props = defineProps({
	variant: {
		type: [Number, String],
		default: 1,
	},
	// Width of the left column. Can be %, fr, px.
	leftWidth: {
		type: String,
		default: "50%",
	},
	// Alignment for both columns: top | center | bottom
	align: {
		type: String,
		default: "top",
	},
	// Specific alignment overrides
	leftAlign: {
		type: String,
		default: null,
	},
	rightAlign: {
		type: String,
		default: null,
	},
	// Optional additional class for the container
	class: {
		type: String,
		default: "",
	},
});

const nav = useNav();
const currentPage = computed(() => nav.currentPage.value);
const showNumber = computed(() => currentPage.value > 1);

const backgroundUrl = computed(() => {
	const mode = isDark.value ? "dark" : "light";
	const variant = props.variant || 1;
	// Keys are relative to this file
	const key = `../assets/images/backgrounds/${mode}/regular-slides/template_${variant}.jpg`;

	const collection = mode === "dark" ? backgroundsDark : backgroundsLight;
	return collection[key] ?? "";
});

// Helper to map alignment to Tailwind classes
function getAlignClass(align) {
	const map = {
		top: "justify-start",
		center: "justify-center",
		bottom: "justify-end",
	};
	return map[align] || map.top;
}

const leftClass = computed(() => getAlignClass(props.leftAlign || props.align));
const rightClass = computed(() =>
	getAlignClass(props.rightAlign || props.align),
);
</script>

<template>
  <div 
    class="slidev-layout two-cols"
    :style="{ backgroundImage: backgroundUrl ? `url(${backgroundUrl})` : 'none', backgroundSize: 'cover', backgroundPosition: 'center' }"
  >
    <div class="flex flex-col h-full gap-8">
      <div v-if="$slots.header">
        <slot name="header" />
      </div>

      <!-- Grid layout: left column with specified width, right column takes remaining space -->
      <div 
        class="w-full grid gap-8 flex-1"
        :style="{ gridTemplateColumns: `${props.leftWidth} 1fr` }"
      >
        <div class="col-left flex flex-col" :class="leftClass">
          <slot />
        </div>
        <div class="col-right flex flex-col" :class="rightClass">
          <slot name="right" />
        </div>
      </div>
    </div>

    <div v-if="showNumber" class="slide-number">
      {{ currentPage }}
    </div>
  </div>
</template>
