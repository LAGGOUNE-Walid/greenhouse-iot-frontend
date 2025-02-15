<script setup lang="ts">
import { ref, watch, computed, onMounted } from 'vue';
import axios from 'axios';
// @ts-ignore
import VueApexCharts from 'vue3-apexcharts';

const chartDataType = ref(0);
const chartData = ref({ series: [], labels: [] });
const measurementTypes = ref<string[]>([]);
const backendUrl = import.meta.env.VITE_BACKEND_URL;
const colors = ['#3C50E0', '#80CAEE', '#FF5733', '#28A745'];

async function fetchChartData(period: number, backendUrl: string) {
  try {
    const response = await axios.get(`${backendUrl}/api/measurements?interval=${period}`);
    const apiData = response.data.data;

    let labels: string[] = [];
    const measurementTypesSet = new Set<string>();
    let seriesData: any[] = [];

    labels = Object.entries(apiData).map(([key]) => key);
    labels.forEach((date) => {
      Object.values(apiData[date]).forEach((m: any) => measurementTypesSet.add(m.measurement_type_string));
    });

    measurementTypes.value = Array.from(measurementTypesSet);

    seriesData = measurementTypes.value.map((type) => ({
      name: `${type}`,
      data: labels.map((date) => {
        const dayData = Object.values(apiData[date]) as any[];
        const measurement = dayData.find((m) => m.measurement_type_string === type);
        return measurement ? measurement.avg_measuremens : 0;
      })
    }));

    chartData.value = { series: seriesData, labels };
  } catch (error) {
    console.error("Error fetching chart data:", error);
  }
}


watch(chartDataType, (newType) => {
  fetchChartData(newType, backendUrl);
});

onMounted(() => {
  fetchChartData(chartDataType.value, backendUrl);
});

const apexOptions = computed(() => ({
  legend: { show: false, position: 'top', horizontalAlign: 'left' },
  colors,
  chart: {
    fontFamily: 'Satoshi, sans-serif',
    height: 335,
    type: 'area',
    dropShadow: { enabled: true, color: '#623CEA14', top: 10, blur: 4, left: 0, opacity: 0.1 },
    toolbar: { show: false }
  },
  xaxis: {
    type: 'category',
    categories: chartData.value.labels,
    axisBorder: { show: false },
    axisTicks: { show: false }
  },
  yaxis: {
    min: 0,
    title: { style: { fontSize: '0px' } }
  },
  stroke: { width: [2, 2], curve: 'straight' },
  dataLabels: { enabled: false }
}));
</script>

<template>
  <div
    class="col-span-12 rounded-sm border border-stroke bg-white px-5 pt-7.5 pb-5 shadow-default dark:border-strokedark dark:bg-boxdark sm:px-7.5 xl:col-span-12">
    <div class="flex flex-wrap items-start justify-between gap-3 sm:flex-nowrap">
      <div class="flex w-full flex-wrap gap-3 sm:gap-5">
        <div v-for="(type, index) in measurementTypes" :key="index" class="flex min-w-47.5">
          <span
            class="mt-1 mr-2 flex h-4 w-full max-w-4 items-center justify-center rounded-full border"
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
      <div class="flex w-full max-w-45 justify-end">
        <div class="inline-flex items-center rounded-md bg-whiter p-1.5 dark:bg-meta-4">
          <button @click="chartDataType = 0" :class="{ 'bg-white shadow-card': chartDataType == 0 }"
            class="rounded  py-1 px-3 text-xs font-medium text-black  hover:bg-white hover:shadow-card dark:bg-boxdark dark:text-white dark:hover:bg-boxdark">
            Today
          </button>
          <button @click="chartDataType = 7" :class="{ 'bg-white shadow-card': chartDataType == 7 }"
            class="rounded  py-1 px-3 text-xs font-medium text-black hover:bg-white hover:shadow-card dark:text-white dark:hover:bg-boxdark">
            Week
          </button>
          <button @click="chartDataType = 30" :class="{ 'bg-white shadow-card': chartDataType == 30 }"
            class="rounded py-1 px-3 text-xs font-medium text-black hover:bg-white hover:shadow-card dark:text-white dark:hover:bg-boxdark">
            Month
          </button>
        </div>
      </div>
    </div>
    <div>
      <div id="chartOne" class="-ml-5">
        <VueApexCharts type="area" height="350" :options="apexOptions" :series="chartData.series" ref="chart" />
      </div>
    </div>
  </div>
</template>
