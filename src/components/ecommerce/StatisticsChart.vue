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
