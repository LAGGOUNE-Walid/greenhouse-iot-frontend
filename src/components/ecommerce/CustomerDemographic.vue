<template>
  <div class="space-y-5 bg-white p-3 rounded">
    <template v-if="loading">
      <div v-for="i in 4" :key="i" class="flex items-center justify-between animate-pulse">
        <div class="flex items-center gap-3">
          <div class="bg-gray-200 dark:bg-gray-800 rounded-full w-10 h-10"></div>
          <div class="h-4 bg-gray-200 dark:bg-gray-800 rounded w-28"></div>
        </div>
        <div class="flex w-full max-w-[140px] items-center gap-3">
          <div class="relative block h-2 w-full max-w-[100px] rounded bg-gray-200 dark:bg-gray-800">
            <div class="absolute left-0 top-0 h-full rounded bg-gray-300 dark:bg-gray-700" style="width: 70%"></div>
          </div>
          <div class="h-4 bg-gray-200 dark:bg-gray-800 rounded w-8"></div>
        </div>
      </div>
    </template>

    <template v-else>
      <div class="flex items-center justify-between" v-for="node in nodes" :key="node.id">
        <div class="flex items-center gap-3">
          <div class="items-center w-full rounded-full max-w-10">
            <img src="/images/nodemcu.png" alt="usa" />
          </div>
          <div>
            <p class="font-semibold text-gray-800 text-theme-sm dark:text-white/90">
              NODE id #{{ node.id }}
            </p>
          </div>
        </div>
        <div class="flex w-full max-w-[140px] items-center gap-3">
          <div class="relative block h-2 w-full max-w-[100px] rounded bg-gray-200 dark:bg-gray-800">
            <div
              class="absolute left-0 top-0 flex h-full items-center justify-center rounded bg-brand-500 text-xs font-medium text-white"
              :style="{ width: node.last_battery_level?.value + '%' }"></div>
          </div>
          <p class="font-medium text-gray-800 text-theme-sm dark:text-white/90">
            {{ node.last_battery_level?.value ?? 0 }}%
          </p>
        </div>
      </div>
    </template>
  </div>

</template>
<script setup>
import { onMounted, ref, onUnmounted } from 'vue'
const backendUrl = import.meta.env.VITE_BACKEND_URL;
const nodes = ref([])
const loading = ref(true) // <--- new

let eventSource = null; // Store current EventSource instance

function getNodes() {
  try {
    if (eventSource) {
      eventSource.close();
    }
    eventSource = new EventSource(`${backendUrl}/api/nodes`);
    eventSource.onmessage = function (event) {
      const apiData = JSON.parse(event.data).data
      nodes.value = apiData;
      loading.value = false; // <--- loading done
    }
    eventSource.onerror = (err) => {
      console.error("EventSource failed:", err);
    };
  } catch (error) {
    console.error(error);
  }
}

onMounted(() => {
  getNodes();
})
onUnmounted(() => {
  if (eventSource) {
    eventSource.close();
  }
});
</script>
