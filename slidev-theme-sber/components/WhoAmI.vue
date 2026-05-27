<template>
  <div class="whoami">
    <div class="whoami__frame">
      <transition name="whoami-fade" mode="out-in">
        <img
          v-if="currentSrc"
          :key="currentSrc"
          :src="currentSrc"
          :alt="alt"
          class="whoami__img"
        />
      </transition>
    </div>
  </div>
  
</template>

<script setup>
import { computed, onBeforeUnmount, onMounted, ref, watch } from "vue";

const props = defineProps({
	images: { type: Array, required: true },
	intervalMs: { type: Number, default: 4000 },
	alt: { type: String, default: "Who am I photo" },
});

const intervalMs = computed(() => props.intervalMs);
const alt = computed(() => props.alt);

const index = ref(0);
const hasImages = computed(
	() => Array.isArray(props.images) && props.images.length > 0,
);

function resolveSrc(src) {
	if (!src) return "";
	// внешние URL и data: URL не трогаем
	if (/^(https?:)?\/\//.test(src) || src.startsWith("data:")) return src;
	const base = import.meta.env?.BASE_URL || "/";
	return src.startsWith("/") ? `${base}${src.slice(1)}` : `${base}${src}`;
}

const resolvedImages = computed(() =>
	hasImages.value ? props.images.map(resolveSrc) : [],
);

const currentSrc = computed(() =>
	hasImages.value
		? resolvedImages.value[index.value % resolvedImages.value.length]
		: "",
);

let timer;

function next() {
	if (!hasImages.value) return;
	index.value = (index.value + 1) % props.images.length;
}

function start() {
	stop();
	if (!hasImages.value) return;
	// предзагрузка следующего
	resolvedImages.value.forEach((src) => {
		const img = new Image();
		img.src = src;
	});
	timer = window.setInterval(next, intervalMs.value);
}

function stop() {
	if (timer) {
		clearInterval(timer);
		timer = undefined;
	}
}

onMounted(start);
onBeforeUnmount(stop);
watch(intervalMs, start);
watch(
	() => props.images,
	() => {
		index.value = 0;
		start();
	},
);
</script>

<style scoped>
.whoami {
  width: 100%;
  max-width: 100%;
  margin: 0 auto;
}

.whoami__frame {
  width: 100%;
  aspect-ratio: 1 / 1; /* квадратное кадрирование */
  overflow: hidden;
  border-radius: 0.5rem;
}

.whoami__img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  object-position: center;
  display: block;
}

.whoami-fade-enter-active,
.whoami-fade-leave-active {
  transition: opacity 0.5s ease;
}

.whoami-fade-enter-from,
.whoami-fade-leave-to {
  opacity: 0;
}
</style>


