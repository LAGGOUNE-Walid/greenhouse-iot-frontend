<template>
  <div
    class="rounded-2xl border border-gray-200 bg-white px-5 pb-5 pt-5 dark:border-gray-800 dark:bg-white/[0.03] sm:px-6 sm:pt-6">
    <div class="flex flex-col gap-5 mb-6 sm:flex-row sm:justify-between">
      <div class="w-full">
        <h3 class="text-lg font-semibold text-gray-800 dark:text-white/90">Statistics</h3>
        <div v-for="(type, index) in measurementTypes" :key="index" class="flex min-w-47.5">
          <span class="mt-1 mr-2 flex h-4 w-full max-w-4 items-center justify-center rounded-full border"
            :style="{ borderColor: colors[index % colors.length] }">
            <span class="block h-2.5 w-full max-w-2.5 rounded-full"
              :style="{ backgroundColor: colors[index % colors.length] }">
            </span>
          </span>
          <div class="w-full">
            <p class="font-semibold" :style="{ color: colors[index % colors.length] }">{{ type }}</p>
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
          <button @click="exportExcel"
            class="inline-flex items-center justify-center font-medium gap-1 rounded-lg transition px-4 py-1 text-sm bg-brand-500 text-white shadow-theme-xs hover:bg-brand-600 disabled:bg-brand-300"><span
              class="flex items-center"><svg xmlns="http://www.w3.org/2000/svg" width="16" height="16"
                fill="currentColor" class="bi bi-file-earmark-spreadsheet" viewBox="0 0 16 16">
                <path
                  d="M14 14V4.5L9.5 0H4a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h8a2 2 0 0 0 2-2M9.5 3A1.5 1.5 0 0 0 11 4.5h2V9H3V2a1 1 0 0 1 1-1h5.5zM3 12v-2h2v2zm0 1h2v2H4a1 1 0 0 1-1-1zm3 2v-2h3v2zm4 0v-2h3v1a1 1 0 0 1-1 1zm3-3h-3v-2h3zm-7 0v-2h3v2z" />
              </svg></span> Export excel</button>
        </div>
      </div>
    </div>
    <div class="max-w-full overflow-x-auto custom-scrollbar">
      <div v-for="(series, index) in chartData.series" :key="index" class="mb-6">
        <h4 class="text-md font-semibold text-gray-700 dark:text-white">{{ series.name }}</h4>
        <VueApexCharts type="area" height="310" :options="apexOptions(series)" :series="[series]" />
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

const selected = ref(0);
const chartData = ref({ series: [], labels: [] });
const measurementTypes = ref([]);
const backendUrl = import.meta.env.VITE_BACKEND_URL;
const colors = ["#3C50E0", "#80CAEE", "#FF5733", "#28A745"];

let refreshInterval; // Store interval ID

function exportExcel() {
  const url = `${backendUrl}/api/measurements-export`;
  fetch(url, {
    method: "GET",
    headers: {
      "Accept": "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet" // Excel MIME type
    }
  })
    .then(response => {
      if (!response.ok) {
        throw new Error("Network response was not ok");
      }
      return response.blob(); // Convert response to a Blob
    })
    .then(blob => {
      const url = window.URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      a.download = "export.xlsx"; // Default filename
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
      window.URL.revokeObjectURL(url);
    })
    .catch(error => console.error("Error downloading file:", error));
}

async function fetchChartData(period) {
  try {
    const response = await axios.get(`${backendUrl}/api/measurements?interval=${period}`);
    const apiData = response.data.data;

    let labels = Object.keys(apiData);
    const measurementTypesSet = new Set();
    let seriesData = [];

    labels.forEach(date => {
      Object.values(apiData[date]).forEach(m => measurementTypesSet.add(m.measurement_type_string));
    });

    measurementTypes.value = Array.from(measurementTypesSet);

    seriesData = measurementTypes.value.map((type, index) => ({
      name: `${type}`,
      data: labels.map(date => {
        const dayData = Object.values(apiData[date]);
        const measurement = dayData.find(m => m.measurement_type_string === type);
        return measurement ? measurement.avg_measuremens : 0;
      }),
      color: colors[index % colors.length]
    }));

    chartData.value = { series: seriesData, labels };
  } catch (error) {
    console.error("Error fetching chart data:", error);
  }
}

// Watch for period change
watch(selected, newType => {
  fetchChartData(newType);
});

// Fetch data initially & set auto-refresh
onMounted(() => {
  fetchChartData(selected.value);

  refreshInterval = setInterval(() => {
    fetchChartData(selected.value);
  }, 1000); // Auto-refresh every second
});

// Cleanup interval when component unmounts
onUnmounted(() => {
  clearInterval(refreshInterval);
});

const apexOptions = (series) => ({
  legend: { show: false },
  colors: [series.color],
  chart: {
    fontFamily: "Satoshi, sans-serif",
    type: "area",
    toolbar: { show: false },
  },
  xaxis: {
    type: "category",
    categories: chartData.value.labels,
    axisBorder: { show: false },
    axisTicks: { show: false },
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
