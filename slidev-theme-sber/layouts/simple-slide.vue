<script setup>
import { useNav } from "@slidev/client";
import { computed } from "vue";
import { isDark } from "@slidev/client/logic/dark.ts";

// Подтягиваем все фоновые картинки
// import.meta.glob возвращает URL-строки для ассетов (картинок/шрифтов и т.п.)
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
	// положение контента: верх / центр / низ (значения: top | center | bottom)
	contentPos: {
		type: String,
		default: "top",
	},
	// выравнивание контента: влево / по центру (значения: left | center)
	contentAlign: {
		type: String,
		default: "left",
	},
});

const nav = useNav();
const currentPage = computed(() => nav.currentPage.value);
const showNumber = computed(() => currentPage.value > 1);

const backgroundUrl = computed(() => {
	const mode = isDark.value ? "dark" : "light";
	const variant = props.variant || 1;
	// Собираем ключ для поиска в результате import.meta.glob
	// Важно: ключи import.meta.glob — относительные к текущему модулю (этому файлу).

	// Мы находимся в layouts/simple-slide.vue,
	// а ассеты лежат в ../assets/images/...
	const key = `../assets/images/backgrounds/${mode}/regular-slides/template_${variant}.jpg`;

	const collection = mode === "dark" ? backgroundsDark : backgroundsLight;
	return collection[key] ?? "";
});

const contentPosClass = computed(() => {
	const map = {
		top: "justify-start",
		center: "justify-center",
		bottom: "justify-end",
	};
	return map[String(props.contentPos)] || map.top;
});

const contentAlignClass = computed(() => {
	return String(props.contentAlign) === "center"
		? "items-center text-center"
		: "items-start text-left";
});
</script>

<template>
  <div 
    class="slidev-layout"
    :style="{ backgroundImage: backgroundUrl ? `url(${backgroundUrl})` : 'none', backgroundSize: 'cover', backgroundPosition: 'center' }"
  >
    <div
      class="w-full h-full flex flex-col"
      :class="[contentPosClass, contentAlignClass]"
    >
      <slot />
    </div>
    <div v-if="showNumber" class="slide-number">
      {{ currentPage }}
    </div>
  </div>
</template>
