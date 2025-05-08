<template>
  <div
    class="rounded-2xl border border-gray-200 bg-white px-5 pb-5 pt-5 dark:border-gray-800 dark:bg-white/[0.03] sm:px-6 sm:pt-6">
    <div class="flex flex-col gap-5 mb-6 sm:flex-row sm:justify-between">
      <div class="w-full">
        <h3 class="text-lg font-semibold text-gray-800 dark:text-white/90">Statistics</h3>
        <div v-for="(type, index) in measurementTypes" :key="index" class="flex min-w-47.5">
          <span class="mt-1 mr-2 flex h-4 w-full max-w-4 items-center justify-center rounded-full border"
            :style="{ borderColor: colorForIndex(index) }">
            <span class="block h-2.5 w-full max-w-2.5 rounded-full" :style="{ backgroundColor: colorForIndex(index) }">
            </span>
          </span>
          <div class="w-full">
            <p class="font-semibold" :style="{ color: colorForIndex(index) }">{{ type }}</p>
          </div>
        </div>
      </div>
      <div class="relative">
        <div class="inline-flex items-center gap-0.5 rounded-lg bg-gray-100 p-0.5 dark:bg-gray-900">
          <button v-for="option in options" :key="option.value" @click="selected = option.value" :class="[
            selected === option.value
              ? 'shadow-theme-xs text-gray-900 dark:text-white bg-white dark:bg-gray-800'
              : 'text-gray-500 dark:text-gray-400',
            'px-3 py-2 font-medium rounded-md text-theme-sm hover:text-gray-900 hover:shadow-theme-xs dark:hover:bg-gray-800 dark:hover:text-white',
          ]">
            {{ option.label }}
          </button>
          <button @click="exportExcel" :disabled="httpLoading"
            class="inline-flex items-center justify-center font-medium gap-1 rounded-lg transition px-4 py-1 text-sm shadow-theme-xs"
            :class="httpLoading
              ? 'bg-brand-300 cursor-not-allowed text-white'
              : 'bg-brand-500 text-white hover:bg-brand-600'">
            <span class="flex items-center"><svg xmlns="http://www.w3.org/2000/svg" width="16" height="16"
                fill="currentColor" class="bi bi-file-earmark-spreadsheet" viewBox="0 0 16 16">
                <path
                  d="M14 14V4.5L9.5 0H4a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h8a2 2 0 0 0 2-2M9.5 3A1.5 1.5 0 0 0 11 4.5h2V9H3V2a1 1 0 0 1 1-1h5.5zM3 12v-2h2v2zm0 1h2v2H4a1 1 0 0 1-1-1zm3 2v-2h3v2zm4 0v-2h3v1a1 1 0 0 1-1 1zm3-3h-3v-2h3zm-7 0v-2h3v2z" />
              </svg></span> Export excel</button>

          &nbsp;
          <div role="status" v-if="httpLoading">
            <svg aria-hidden="true" class="w-8 h-8 text-gray-200 animate-spin dark:text-gray-600 fill-blue-600"
              viewBox="0 0 100 101" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path
                d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z"
                fill="currentColor" />
              <path
                d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z"
                fill="currentFill" />
            </svg>
            <span class="sr-only">Loading...</span>
          </div>


        </div>
      </div>
    </div>
    <div class="max-w-full overflow-x-auto custom-scrollbar">
      <div v-for="(series, index) in chartData.series" :key="index" class="mb-6">
        <h4 class="text-md font-semibold text-gray-700 dark:text-white">{{ series.name }}</h4>
        <VueApexCharts type="line" height="310" :options="apexOptions(series)" :series="[series]" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, computed, onMounted, onUnmounted } from "vue";
import VueApexCharts from 'vue3-apexcharts';
import axios from "axios";

const options = [
  { value: 0, label: 'Today' },
  { value: 7, label: 'Week' },
  { value: 30, label: 'Month' },
];

const httpLoading = ref(false);
const selected = ref(0);
const chartData = ref({ series: [], labels: [] });
const measurementTypes = ref([]);
const backendUrl = import.meta.env.VITE_BACKEND_URL;
const colors = ["#3C50E0", "#80CAEE", "#FF5733", "#28A745"];

let refreshInterval; // Store interval ID

const colorForIndex = (index) => colors[index % colors.length];

const exportExcel = async () => {
  try {
    httpLoading.value = true;
    const response = await fetch(`${backendUrl}/api/measurements-export`, {
      headers: {
        Accept: "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
      }
    });

    if (!response.ok) throw new Error("Failed to download Excel.");

    const blob = await response.blob();
    const url = window.URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = "export.xlsx";
    a.click();
    window.URL.revokeObjectURL(url);
  } catch (err) {
    console.error("Export failed", err);
  } finally {
    httpLoading.value = false;
  }
};

let eventSource = null;
let reconnectTimeout = null;
const reconnectDelay = 5000; // 5 seconds

function fetchChartData(period) {
  httpLoading.value = true;

  if (eventSource) {
    eventSource.close();
    eventSource = null;
  }

  const connect = () => {
    eventSource = new EventSource(`${backendUrl}/api/measurements?interval=${period}`);

    eventSource.onmessage = (event) => {
      httpLoading.value = false;

      const apiData = JSON.parse(event.data).data;
      const labels = Object.keys(apiData);
      const measurementTypesSet = new Set();

      labels.forEach(date => {
        Object.values(apiData[date]).forEach(m => measurementTypesSet.add(m.measurement_type_string));
      });

      measurementTypes.value = Array.from(measurementTypesSet);

      const seriesData = measurementTypes.value.map((type, index) => ({
        name: type,
        data: labels.map(date => {
          const dayData = Object.values(apiData[date]);
          const measurement = dayData.find(m => m.measurement_type_string === type);
          return measurement ? measurement.avg_measuremens : 0;
        }),
        color: colors[index % colors.length]
      }));

      chartData.value = { series: seriesData, labels };
    };

    eventSource.onerror = (err) => {
      console.error("EventSource error:", err);
      httpLoading.value = false;

      // Cleanup and retry connection
      eventSource.close();
      eventSource = null;

      // Prevent stacking reconnects
      if (!reconnectTimeout) {
        reconnectTimeout = setTimeout(() => {
          console.log("Reconnecting to EventSource...");
          connect();
          reconnectTimeout = null;
        }, reconnectDelay);
      }
    };
  };

  connect();
}



// Watch for period change
watch(selected, async (newType) => {
  httpLoading.value = true;
  fetchChartData(newType);
  httpLoading.value = false;
});

// Fetch data initially & set auto-refresh
onMounted(() => {
  fetchChartData(selected.value);

  // refreshInterval = setInterval(() => {
  //   fetchChartData(selected.value);
  // }, 3000); // Auto-refresh every second
});

// Cleanup interval when component unmounts
onUnmounted(() => {
  if (eventSource) {
    eventSource.close();
  }
  if (reconnectTimeout) {
    clearTimeout(reconnectTimeout);
  }
});


const apexOptions = (series) => ({
  legend: { show: false },
  colors: [series.color],
  chart: {
    fontFamily: "Satoshi, sans-serif",
    type: "line",
    toolbar: { show: false },
  },
  xaxis: {
    type: "category",
    categories: chartData.value.labels,
    axisBorder: { show: false },
    // axisTicks: { show: false },
    tickAmount: 5
  },
  yaxis: {
    min: 0,
  },
  stroke: { width: 2, curve: "straight" },
  dataLabels: { enabled: false },
});
</script>


<style scoped>
.area-chart {
  width: 100%;
}
</style>
