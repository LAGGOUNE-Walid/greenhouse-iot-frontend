<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const backendUrl = import.meta.env.VITE_BACKEND_URL; 

const cardItems = ref([]);
const isLoading = ref(true);
let eventSource = null;

function fetchCardsData() {
  try {
    if (eventSource) eventSource.close();

    eventSource = new EventSource(`${backendUrl}/api/measurements?interval=-1`);

    eventSource.onmessage = function (event) {
      const apiData = JSON.parse(event.data).data;
      isLoading.value = false;

      cardItems.value = [
        {
          icon: '🌡️',
          title: 'Temperature',
          total: `${apiData.temperature.value} °C`,
          node_id: apiData.temperature.node_id,
          created_at: apiData.temperature.created_at,
        },
        {
          icon: '💧',
          title: 'Humidity',
          total: `${apiData.humidity.value} %`,
          node_id: apiData.humidity.node_id,
          created_at: apiData.humidity.created_at,
        },
        {
          icon: '🗜',
          title: 'Pressure',
          total: `${apiData.pressure.value} hPa`,
          node_id: apiData.pressure.node_id,
          created_at: apiData.pressure.created_at,
        }
      ];

      apiData.soil_moisture.forEach(sm => {
        cardItems.value.push({
          icon: '🪴',
          title: 'Soil Moisture',
          total: `${sm.value} %`,
          node_id: sm.node_id,
          created_at: sm.created_at,
        });
      });
    };

    eventSource.onerror = (err) => {
      console.error("EventSource failed:", err);
      isLoading.value = false;
    };
  } catch (error) {
    console.error("Error fetching data:", error);
    isLoading.value = false;
  }
}

onMounted(() => {
  fetchCardsData();
});

onUnmounted(() => {
  if (eventSource) eventSource.close();
});
</script>

<template>
  <div class="flex flex-wrap gap-4">
    <!-- Shimmer cards -->
    <template v-if="isLoading">
      <div v-for="n in 3" :key="n" class="flex-1 min-w-[250px] rounded-sm border border-stroke bg-white py-6 px-7.5 shadow-default dark:border-strokedark dark:bg-boxdark animate-pulse">
        <div class="h-11.5 w-11.5 rounded-full bg-slate-200 dark:bg-slate-700"></div>
        <div class="mt-4 space-y-2">
          <div class="h-5 w-3/4 bg-slate-200 dark:bg-slate-700 rounded"></div>
          <div class="h-4 w-1/2 bg-slate-200 dark:bg-slate-700 rounded"></div>
          <div class="h-4 w-1/3 bg-slate-200 dark:bg-slate-700 rounded"></div>
          <div class="h-4 w-1/4 bg-slate-200 dark:bg-slate-700 rounded"></div>
        </div>
      </div>
    </template>

    <!-- Actual cards -->
    <template v-else>
      <div v-for="(item, index) in cardItems" :key="index"
        class="flex-1 min-w-[250px] rounded-sm border border-stroke bg-white py-6 px-7.5 shadow-default dark:border-strokedark dark:bg-boxdark">
        <div class="flex h-11.5 w-11.5 items-center justify-center rounded-full bg-meta-2 dark:bg-meta-4 text-2xl">
          {{ item.icon }}
        </div>
        <div class="mt-4 flex items-end justify-between">
          <div>
            <h4 class="text-title-md font-bold text-black dark:text-white">{{ item.total }}</h4>
            <span class="text-sm font-medium">{{ item.title }}</span><br />
            <span class="text-sm font-medium">Node id #{{ item.node_id }}</span><br />
            <span class="text-sm font-medium">At {{ item.created_at }}</span>
          </div>
        </div>
      </div>
    </template>
  </div>
</template>

<style scoped>
/* Add basic pulse animation if Tailwind doesn't have it set */
@keyframes pulse {
  0%, 100% { opacity: 1 }
  50% { opacity: 0.4 }
}
.animate-pulse {
  animation: pulse 1.5s ease-in-out infinite;
}
</style>
