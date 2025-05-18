<template>
    <div>
        <apexchart width="100%" height="400" type="line" :options="chartOptions" :series="series" />
    </div>
</template>

<script setup>
import { onMounted, ref } from 'vue'
import ApexChart from 'vue3-apexcharts'
const backendUrl = import.meta.env.VITE_BACKEND_URL;
const series = ref([])
const chartOptions = ref({
    chart: {
        id: 'measurements-chart',
        type: 'line',
        zoom: { enabled: true },
    },
    xaxis: {
        type: 'datetime',
        title: { text: 'Date' }
    },
    yaxis: {
        title: { text: 'Average Measurement' }
    },
    stroke: {
        curve: 'smooth'
    },
    tooltip: {
        x: {
            format: 'yyyy-MM-dd'
        }
    },
    legend: {
        position: 'top'
    }
})

onMounted(async () => {
    try {

        const response = await fetch(`${backendUrl}/api/measurements?static=1&all=1`)
        const json = await response.json()
        const { data } = json

        const typeMap = {}

        for (const [date, values] of Object.entries(data)) {
            const timestamp = new Date(date).getTime()

            values.forEach(entry => {
                const type = entry.measurement_type_string
                const avg = entry.avg_measuremens

                if (!typeMap[type]) typeMap[type] = []
                typeMap[type].push({ x: timestamp, y: avg })
            })
        }

        series.value = Object.entries(typeMap).map(([type, data]) => ({
            name: type,
            data
        }))
    } catch (error) {
        console.error('Failed to fetch chart data:', error)
    }
})
</script>