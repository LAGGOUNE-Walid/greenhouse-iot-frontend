<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import axios from 'axios';

const backendUrl = import.meta.env.VITE_BACKEND_URL;
const cardItems = ref([]);
const lastUpdated = ref('');
let refreshInterval;

async function fetchCardsData() {
    try {
        const response = await axios.get(`${backendUrl}/api/measurements?interval=-1`);
        const apiData = response.data.data;
        lastUpdated.value = new Date().toLocaleTimeString();

        cardItems.value = [
            {
                icon: `<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-moisture" viewBox="0 0 16 16">...</svg>`,
                title: 'Soil Moisture',
                total: `${apiData.soil_moisture.value} %`,
                created_at: `${apiData.soil_moisture.created_at}`
            },
            {
                icon: `<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-thermometer-sun" viewBox="0 0 16 16">...</svg>`,
                title: 'Temperature',
                total: `${apiData.temperature.value} °C`,
                created_at: `${apiData.temperature.created_at}`
            },
            {
                icon: `<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-droplet" viewBox="0 0 16 16">...</svg>`,
                title: 'Humidity',
                total: `${apiData.humidity.value} %`,
                created_at: `${apiData.humidity.created_at}`
            },
            {
                icon: `<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-speedometer" viewBox="0 0 16 16">...</svg>`,
                title: 'Pressure',
                total: `${apiData.pressure.value} hPa`,
                created_at: `${apiData.pressure.created_at}`
            }
        ];
    } catch (error) {
        console.error("Error fetching chart data:", error);
    }
}

onMounted(() => {
    fetchCardsData();
    refreshInterval = setInterval(fetchCardsData, 1000); // Auto-refresh every second
});

onUnmounted(() => {
    clearInterval(refreshInterval);
});
</script>

<template>
    <!-- Card Item Start -->
    <div v-for="(item, index) in cardItems" :key="index"
        class="flex-1 min-w-[250px] rounded-sm border border-stroke bg-white py-6 px-7.5 shadow-default dark:border-strokedark dark:bg-boxdark">
        <div class="flex h-11.5 w-11.5 items-center justify-center rounded-full bg-meta-2 dark:bg-meta-4"
            v-html="item.icon"></div>

        <div class="mt-4 flex items-end justify-between">
            <div>
                <h4 class="text-title-md font-bold text-black dark:text-white">{{ item.total }}</h4>
                <span class="text-sm font-medium">{{ item.title }}</span>
                <br />
                <span class="text-sm font-medium">At {{ item.created_at }}</span>
            </div>
        </div>
    </div>
    <!-- Card Item End -->
</template>