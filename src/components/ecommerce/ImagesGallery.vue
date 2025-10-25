<template>
  <div class="p-4 max-w-7xl mx-auto">
    <h2 class="text-xl font-semibold mb-4">Image Gallery</h2>

    <template v-if="loading">
      <div class="grid grid-cols-5 gap-4">
        <div
          v-for="n in 10"
          :key="'loading-' + n"
          class="h-32 bg-gray-300 rounded animate-shimmer"
        ></div>
      </div>
    </template>

    <template v-else-if="Object.keys(groupedImages).length">
      <div
        v-for="(group, folderName) in groupedImages"
        :key="folderName"
        class="mb-6 border border-gray-300 rounded"
      >
        <div
          class="cursor-pointer bg-gray-100 p-3 font-semibold flex justify-between items-center"
          @click="toggleFolder(folderName)"
        >
          {{ folderName }}
          <span class="text-sm text-gray-500">{{ openedFolders[folderName] ? '▲' : '▼' }}</span>
        </div>

        <div
          v-if="openedFolders[folderName]"
          class="grid grid-cols-5 gap-4 p-4 border-t"
        >
          <div v-for=" (img,index) in group" :key="img.id" class="flex flex-col items-center">
            <img
              :src="backendUrl + '/storage/' + img.path"
              alt="Gallery image"
              class="object-cover w-full h-32 cursor-pointer rounded shadow hover:scale-105 transition-transform duration-200"
              @click="openModal(img, group[index+1], index+1, folderName)"
              loading="lazy"
            />
            <span class="text-xs text-gray-500 mt-1">
              {{ format(parseISO(img.created_at), "dd MMM yyyy H:I:s") }}
            </span>
          </div>
        </div>
      </div>
    </template>

    <p v-else class="text-center text-gray-500">No images found.</p>

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
        <span >{{  selectedImage.id  }}</span>
        <button @click="moveToNext"><h2>NEXT PIC</h2></button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
import { parseISO, format, getYear, getMonth, getDate, startOfMonth } from "date-fns";

const backendUrl = import.meta.env.VITE_BACKEND_URL;
const images = ref([]);
const loading = ref(true);
const modalOpen = ref(false);
const selectedImage = ref(null);
const nextImage = ref(null);
const folderName = ref(null);
const nextLoopIndex = ref(null);
const openedFolders = ref({});

function toggleFolder(name) {
  openedFolders.value[name] = !openedFolders.value[name];
}

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

// Calculate week number of the month
function getWeekOfMonth(date) {
  const dayOfMonth = getDate(date);
  const firstDayOfMonth = startOfMonth(date).getDay(); // 0 (Sun) - 6 (Sat)
  return Math.ceil((dayOfMonth + firstDayOfMonth) / 7);
}

const groupedImages = computed(() => {
  const groups = {};
  for (const img of images.value) {
    const date = parseISO(img.created_at);
    const weekInMonth = getWeekOfMonth(date);
    const folder = `Week ${weekInMonth} - ${format(date, "MMMM")} - ${getYear(date)}`;
    if (!groups[folder]) {
      groups[folder] = [];
      openedFolders.value[folder] = false;
    }
    groups[folder].push(img);
  }
  return groups;
});

function  moveToNext() {
  selectedImage.value = nextImage.value;
  nextImage.value = groupedImages.value[folderName.value][nextLoopIndex.value+1]

  nextLoopIndex.value = nextLoopIndex.value + 1
  modalOpen.value = true;

}


function openModal(img, nextImg, nextIndex, fName) {
  nextImage.value = nextImg;
  folderName.value = fName;
  nextLoopIndex.value = nextIndex;
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
