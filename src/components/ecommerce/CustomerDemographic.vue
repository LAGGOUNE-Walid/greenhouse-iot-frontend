<template>
  <div class="rounded-2xl border border-gray-200 bg-white p-5 dark:border-gray-800 dark:bg-white/[0.03] sm:p-6">
    <div class="space-y-5">
      <div class="flex items-center justify-between" v-for="node in nodes" :key="node.id">
        <div class="flex items-center gap-3">
          <div class="items-center w-full rounded-full max-w-10">
            <img src="/images/nodemcu.png" alt="usa" />
          </div>
          <div>
            <p class="font-semibold text-gray-800 text-theme-sm dark:text-white/90">NODE id #{{ node.id }}</p>
          </div>
        </div>
        <div class="flex w-full max-w-[140px] items-center gap-3">
          <div class="relative block h-2 w-full max-w-[100px] rounded bg-gray-200 dark:bg-gray-800">
            <div
              class="absolute left-0 top-0 flex h-full items-center justify-center rounded bg-brand-500 text-xs font-medium text-white"
              :style="{ 'width' : node.last_battery_level?.value+'%' }"
            ></div>
          </div>
          <p class="font-medium text-gray-800 text-theme-sm dark:text-white/90">{{ node.last_battery_level?.value ?? 0
            }}%</p>
        </div>
      </div>
    </div>
  </div>
</template>
<script setup>
import { onMounted, ref, onUnmounted } from 'vue'
const backendUrl = import.meta.env.VITE_BACKEND_URL;
const nodes = ref([])
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
