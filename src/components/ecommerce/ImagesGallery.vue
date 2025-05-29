<template>
  <div class="p-4 max-w-7xl mx-auto">
    <h2 class="text-xl font-semibold mb-4">Image Gallery</h2>

    <div class="grid grid-cols-5 gap-4">
      <template v-if="loading">
        <!-- Show 10 shimmer placeholders while loading -->
        <div
          v-for="n in 10"
          :key="'loading-' + n"
          class="h-32 bg-gray-300 rounded animate-shimmer"
        ></div>
      </template>

      <template v-else-if="images.length">
        <img
          v-for="img in images"
          :key="img.id"
          :src="backendUrl + '/storage/' + img.path"
          alt="Gallery image"
          class="object-cover w-full h-32 cursor-pointer rounded shadow hover:scale-105 transition-transform duration-200"
          @click="openModal(img)"
          loading="lazy"
        />
      </template>

      <p v-else class="col-span-5 text-center text-gray-500">No images found.</p>
    </div>

    <!-- Modal -->
    <div
      v-if="modalOpen"
      @click.self="closeModal"
      class="fixed inset-0 bg-black bg-opacity-70 flex items-center justify-center z-50"
    >
      <div class="relative max-w-4xl max-h-[80vh] p-4 bg-white rounded shadow-lg">
        <button
          @click="closeModal"
          class="absolute top-2 right-2 text-gray-700 hover:text-gray-900 text-2xl font-bold"
          aria-label="Close modal"
        >
          &times;
        </button>
        <img
          :src="backendUrl + '/storage/' + selectedImage.path"
          alt="Zoomed image"
          class="max-w-full max-h-[70vh] rounded"
        />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

const backendUrl = import.meta.env.VITE_BACKEND_URL;
const images = ref([]);
const loading = ref(true);
const modalOpen = ref(false);
const selectedImage = ref(null);

async function fetchImages() {
  try {
    const res = await fetch(backendUrl + "/api/images");
    if (!res.ok) throw new Error("Failed to fetch images");
    images.value = await res.json();
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
}

function openModal(img) {
  selectedImage.value = img;
  modalOpen.value = true;
}

function closeModal() {
  modalOpen.value = false;
  selectedImage.value = null;
}

onMounted(() => {
  fetchImages();
});
</script>

<style scoped>
@keyframes shimmer {
  0% {
    background-position: -200px 0;
  }
  100% {
    background-position: 200px 0;
  }
}

.animate-shimmer {
  background: linear-gradient(
    90deg,
    #e0e0e0 25%,
    #f8f8f8 50%,
    #e0e0e0 75%
  );
  background-size: 400% 100%;
  animation: shimmer 1.5s infinite;
}
</style>
