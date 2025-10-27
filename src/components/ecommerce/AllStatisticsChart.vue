<template>
    <div class="space-y-8">
        <div class="flex justify-between items-center mb-4">
            <h2 class="text-xl font-bold">Measurement Dashboard</h2>
            <div class="flex items-center space-x-2">
                <label for="datepicker" class="text-sm font-medium">Select Date:</label>
                <input type="date" id="datepicker" v-model="selectedDate" class="border rounded px-3 py-1"
                    @change="fetchData" />
                <button @click="resetDate" class="bg-gray-200 hover:bg-gray-300 px-3 py-1 rounded text-sm">
                    Show All
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
            </div>
        </div>

        <div v-for="(dataPoints, type) in typeMap" :key="type" class="border rounded p-4 bg-white">
            <h3 class="text-lg font-semibold mb-4">{{ type }}</h3>
            <apexchart width="100%" height="300" type="line" :options="getChartOptions(type, selectedDate)"
                :series="getSeriesConfig(type, dataPoints)" @zoomed="onZoom" @beforeResetZoom="onResetZoom" />
        </div>

        <!-- Battery Levels Chart -->
        <div v-if="Object.keys(batteryData).length > 0" class="border rounded p-4 bg-white">
            <h3 class="text-lg font-semibold mb-4">Battery Levels</h3>
            <apexchart width="100%" height="400" type="line" :options="getChartOptions('battery', selectedDate)"
                :series="getBatterySeriesConfig()" />
        </div>

    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import ApexChart from 'vue3-apexcharts'

const backendUrl = import.meta.env.VITE_BACKEND_URL
const typeMap = ref({})
const batteryData = ref({})
const selectedDate = ref('')
const currentZoom = ref(false)


const discreteDataTypes = []; // These get step lines
const continuousDataTypes = ['soil_moisture', 'water_level', 'temperature', 'humidity', 'pressure', 'battery']; // These get step lines
const units = { 'soil_moisture': "%", 'temperature': "°C", "humidity": "%", 'pressure': "hPa", 'battery': "%" }

function isContinuousDataType(type) {
    return continuousDataTypes.includes(type.toLowerCase());
}

// function getUnit(type)

function getChartOptions(type, dateSelected) {
    const isDailyView = dateSelected !== '';
    const isContinuous = isContinuousDataType(type);
    const isBattery = type.toLowerCase() === 'battery';
    
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
            title: { text: `${type} ${units[type] || ''}` },
            forceNiceScale: true,
            ...(isBattery && { min: 0, max: 100 }),
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
                formatter: function (value) {
                    return value === null ? 'No data' :
                        `${value}${units[type] || ''}`;
                }
            }
        },
        legend: {
            show: isBattery,
            position: 'top',
            horizontalAlign: 'center',
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
     const formattedData = dataPoints.map(point => {
        return [point.x * 1000, point.y];
    });
    return [{
        name: type,
        data: formattedData,
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

function getBatterySeriesConfig() {
    return Object.entries(batteryData.value).map(([nodeId, dataPoints]) => {
        return getSeriesConfig('battery', dataPoints.map(point => ({ x: point.x, y: point.y })))[0];
    }).map((series, index) => ({
        ...series,
        name: `Node ${Object.keys(batteryData.value)[index]}`
    }));
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

        // Fetch battery data
        const batteryResponse = await fetch(`${backendUrl}/api/batteries?chart=1`);
        const batteryJson = await batteryResponse.json();
        batteryData.value = batteryJson;
    } catch (error) {
        console.error('Failed to fetch chart data:', error);
    }
}

function resetDate() {
    selectedDate.value = '';
    fetchData();
}

const httpLoading = ref(false);
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

onMounted(fetchData);
</script>

<style scoped>
/* Optional styling */
</style>