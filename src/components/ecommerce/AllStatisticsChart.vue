<template>
    <div class="space-y-8">
        <div class="flex justify-between items-center mb-4">
            <h2 class="text-xl font-bold">Measurement Dashboard</h2>
            <div class="flex items-center space-x-2">
                <label for="datepicker" class="text-sm font-medium">Select Date:</label>
                <input 
                    type="date" 
                    id="datepicker" 
                    v-model="selectedDate"
                    class="border rounded px-3 py-1"
                    @change="fetchData"
                />
                <button 
                    @click="resetDate"
                    class="bg-gray-200 hover:bg-gray-300 px-3 py-1 rounded text-sm"
                >
                    Show All
                </button>
            </div>
        </div>

        <div v-for="(dataPoints, type) in typeMap" :key="type" class="border rounded p-4 bg-white">
            <h3 class="text-lg font-semibold mb-4">{{ type }}</h3>
            <apexchart 
                width="100%" 
                height="300" 
                type="line" 
                :options="getChartOptions(type, selectedDate)"
                :series="getSeriesConfig(type, dataPoints)" 
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
const selectedDate = ref('')
const currentZoom = ref(false)

// Define which measurement types should use different chart styles
const continuousDataTypes = ['temperature', 'humidity']; // These get smooth lines
const discreteDataTypes = ['soil_moisture', 'water_level']; // These get step lines

function isContinuousDataType(type) {
    return continuousDataTypes.includes(type.toLowerCase());
}

function getChartOptions(type, dateSelected) {
    const isDailyView = dateSelected !== '';
    const isContinuous = isContinuousDataType(type);
    
    return {
        chart: {
            id: `chart-${type}`,
            type: 'line',
            zoom: { enabled: true },
            animations: { enabled: false }
        },
        xaxis: {
            type: 'datetime',
            title: { text: 'Time' },
            labels: {
                datetimeFormatter: isDailyView 
                    ? { hour: 'HH:mm' } 
                    : {
                        year: 'yyyy',
                        month: 'MMM dd',
                        day: 'MMM dd',
                        hour: 'HH:mm',
                    },
                datetimeUTC: false,
            },
            tickAmount: isDailyView ? 24 : undefined,
        },
        yaxis: {
            title: { text: `${type}${isContinuous ? '' : ' (%)'}` },
            min: isContinuous ? undefined : 0,
            max: isContinuous ? undefined : 100,
            forceNiceScale: true,
        },
        stroke: {
            curve: isContinuous ? 'smooth' : 'stepline',
            width: 2,
        },
        markers: {
            size: isContinuous ? 0 : 4,
            hover: {
                sizeOffset: 2
            }
        },
        tooltip: {
            x: {
                format: isDailyView ? 'HH:mm' : 'dd MMM yyyy HH:mm',
            },
            y: {
                formatter: function(value) {
                    return value === null ? 'No data' : 
                           isContinuous ? value : `${value}%`;
                }
            }
        },
        legend: {
            show: false,
        },
        dataLabels: {
            enabled: false,
        },
        noData: {
            text: 'No data available',
            align: 'center',
            verticalAlign: 'middle',
        },
    };
}

function getSeriesConfig(type, dataPoints) {
    const isContinuous = isContinuousDataType(type);
    
    return [{
        name: type,
        data: dataPoints,
        ...(!isContinuous && {
            strokeDashArray: [5, 5],
            fill: {
                type: 'gradient',
                gradient: {
                    shadeIntensity: 1,
                    opacityFrom: 0.7,
                    opacityTo: 0.3,
                }
            }
        })
    }];
}

function onZoom(chartCtx, { xaxis }) {
    const range = xaxis.max - xaxis.min;
    const newZoom = range < 3 * 24 * 60 * 60 * 1000;
    if (newZoom !== currentZoom.value) {
        currentZoom.value = newZoom;
        chartCtx.updateOptions({
            tooltip: {
                x: { 
                    format: newZoom ? 'dd MMM yyyy HH:mm' : 'yyyy-MM-dd' 
                },
            },
            xaxis: {
                labels: {
                    datetimeFormatter: newZoom
                        ? { hour: 'HH:mm', minute: 'HH:mm' }
                        : { day: 'MMM dd', month: 'MMM dd', year: 'yyyy' },
                },
            },
        });
    }
}

function onResetZoom() {
    currentZoom.value = false;
}

async function fetchData() {
    try {
        const params = new URLSearchParams();
        params.append('static', 1);
        params.append('all', 1);
        if (selectedDate.value) {
            params.append('date', selectedDate.value);
        }
        
        const response = await fetch(`${backendUrl}/api/measurements?${params.toString()}`);
        const json = await response.json();
        typeMap.value = json.data;
    } catch (error) {
        console.error('Failed to fetch chart data:', error);
    }
}

function resetDate() {
    selectedDate.value = '';
    fetchData();
}

onMounted(fetchData);
</script>

<style scoped>
/* Optional styling */
</style>