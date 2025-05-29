<template>
  <div class="space-y-8">
    <div
      v-for="(dataPoints, type) in typeMap"
      :key="type"
      class="border rounded p-4 bg-white"
    >
      <h3 class="text-lg font-semibold mb-4">{{ type }}</h3>
      <apexchart
        width="100%"
        height="300"
        type="line"
        :options="getChartOptions(type)"
        :series="[{ name: type, data: dataPoints }]"
        @zoomed="onZoom"
        @beforeResetZoom="onResetZoom"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import ApexChart from 'vue3-apexcharts'

const backendUrl = import.meta.env.VITE_BACKEND_URL
const typeMap = ref({})

// Track zoom state per type (optional, here we just adjust global chart options)
const currentZoom = ref(false)

function getChartOptions(type) {
  return {
    chart: {
      id: `chart-${type}`,
      type: 'line',
      zoom: { enabled: true },
    },
    xaxis: {
      type: 'datetime',
      title: { text: 'Date' },
      labels: {
        datetimeFormatter: {
          year: 'yyyy',
          month: 'MMM dd',
          day: 'MMM dd',
          hour: 'HH:mm',
          minute: 'HH:mm',
        },
      },
    },
    yaxis: {
      title: { text: `${type} (Average)` },
    },
    stroke: {
      curve: 'smooth',
    },
    tooltip: {
      x: {
        format: currentZoom.value ? 'dd MMM yyyy HH:mm' : 'yyyy-MM-dd',
      },
    },
    legend: {
      show: false,
    },
  }
}

// Update tooltip & labels format dynamically on zoom in/out
function onZoom(chartCtx, { xaxis }) {
  const range = xaxis.max - xaxis.min
  // If zoom range < 3 days, show detailed datetime; else show date only
  currentZoom.value = range < 3 * 24 * 60 * 60 * 1000
}

function onResetZoom() {
  currentZoom.value = false
}

onMounted(async () => {
  try {
    const response = await fetch(`${backendUrl}/api/measurements?static=1&all=1`)
    const json = await response.json()
    const { data } = json

    // Expect data to be like:
    // {
    //   "temperature": [{ x: timestamp_ms, y: value }, ...],
    //   "humidity": [{ x: timestamp_ms, y: value }, ...],
    // }

    typeMap.value = data
  } catch (error) {
    console.error('Failed to fetch chart data:', error)
  }
})
</script>

<style scoped>
/* Optional styling */
</style>
