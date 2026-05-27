<script setup>
import { computed } from "vue";
import { isDark } from "@slidev/client/logic/dark.ts";

// Подтягиваем все фоновые картинки
const backgroundsDark = import.meta.glob(
	"../assets/images/backgrounds/dark/title/*.jpg",
	{ eager: true, query: "?url", import: "default" },
);
const backgroundsLight = import.meta.glob(
	"../assets/images/backgrounds/light/title/*.jpg",
	{ eager: true, query: "?url", import: "default" },
);

const props = defineProps({
	variant: {
		type: [Number, String],
		default: 1,
	},
	// положение контента: верх / центр / низ (значения: top | center | bottom)
	contentPos: {
		type: String,
		default: "center",
	},
	// выравнивание контента: влево / по центру (значения: left | center)
	contentAlign: {
		type: String,
		default: "center",
	},
});

const contentPosClass = computed(() => {
	const map = {
		top: "justify-start",
		center: "justify-center",
		bottom: "justify-end",
	};
	return map[String(props.contentPos)] || map.center;
});

const contentAlignClass = computed(() => {
	return String(props.contentAlign) === "left"
		? "items-start text-left"
		: "items-center text-center";
});

const backgroundUrl = computed(() => {
	const mode = isDark.value ? "dark" : "light";
	const variant = props.variant || 1;

	const key = `../assets/images/backgrounds/${mode}/title/title_${variant}.jpg`;

	const collection = mode === "dark" ? backgroundsDark : backgroundsLight;
	return collection[key] ?? "";
});
</script>

<template>
  <div 
    class="slidev-layout title"
    :style="{ backgroundImage: backgroundUrl ? `url(${backgroundUrl})` : 'none', backgroundSize: 'cover', backgroundPosition: 'center' }"
  >
    <div
      class="w-full h-full flex flex-col"
      :class="[contentPosClass, contentAlignClass]"
    >
      <slot />
    </div>
  </div>
</template>