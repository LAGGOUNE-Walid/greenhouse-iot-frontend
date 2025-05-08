<script setup>
import { ref, onMounted, nextTick } from 'vue'
import axios from 'axios'

const measurements = ref([])
const pagination = ref(null)
const loading = ref(false)
const tableContainer = ref(null)
const backendUrl = import.meta.env.VITE_BACKEND_URL;
const fetchMeasurements = async (url = `${backendUrl}/api/measurements-table`) => {
  loading.value = true
  try {
    const response = await axios.get(url)
    measurements.value = response.data.data
    pagination.value = response.data
    await nextTick()
  } catch (error) {
    console.error('Failed to fetch measurements:', error)
  } finally {
    loading.value = false
  }
}
const changePage = (url) => {
  if (url) fetchMeasurements(url)
}

onMounted(() => {
  fetchMeasurements()
})
</script>

<template>
  <div ref="tableContainer" class="overflow-x-auto">
    <!-- SHIMMER PLACEHOLDER -->
    <div v-if="loading">
      <table class="min-w-full animate-pulse">
        <thead>
          <tr class="bg-gray-100">
            <th class="px-6 py-3 text-left text-sm font-medium text-gray-500">ID</th>
            <th class="px-6 py-3 text-left text-sm font-medium text-gray-500">Value</th>
            <th class="px-6 py-3 text-left text-sm font-medium text-gray-500">Node ID</th>
            <th class="px-6 py-3 text-left text-sm font-medium text-gray-500">Type</th>
            <th class="px-6 py-3 text-left text-sm font-medium text-gray-500">Created At</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="n in 10" :key="n" class="bg-white border-b">
            <td class="px-6 py-4">
              <div class="h-4 bg-gray-300 rounded w-3/4"></div>
            </td>
            <td class="px-6 py-4">
              <div class="h-4 bg-gray-300 rounded w-2/4"></div>
            </td>
            <td class="px-6 py-4">
              <div class="h-4 bg-gray-300 rounded w-1/3"></div>
            </td>
            <td class="px-6 py-4">
              <div class="h-4 bg-gray-300 rounded w-2/3"></div>
            </td>
            <td class="px-6 py-4">
              <div class="h-4 bg-gray-300 rounded w-5/6"></div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- REAL TABLE -->
    <table v-else class="min-w-full text-sm text-left text-gray-700 bg-white shadow rounded-md">
      <thead class="text-xs text-gray-700 uppercase bg-gray-100">
        <tr>
          <th class="px-6 py-3">ID</th>
          <th class="px-6 py-3">Value</th>
          <th class="px-6 py-3">Node ID</th>
          <th class="px-6 py-3">Type</th>
          <th class="px-6 py-3">Created At</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in measurements" :key="item.id" class="border-b hover:bg-gray-50">
          <td class="px-6 py-4">{{ item.id }}</td>
          <td class="px-6 py-4">{{ item.value }}</td>
          <td class="px-6 py-4">{{ item.node_id }}</td>
          <td class="px-6 py-4">{{ item.measurement_type_string }}</td>
          <td class="px-6 py-4">{{ item.created_at }}</td>
        </tr>
      </tbody>
    </table>
    <!-- PAGINATION -->
  <div v-if="pagination" class="flex justify-center items-center mt-6 space-x-2 flex-wrap">
    <button
      v-for="link in pagination.meta.links"
      :key="link.label"
      :disabled="!link.url || link.active"
      @click="changePage(link.url)"
      class="px-3 py-1 rounded-md text-sm"
      :class="[
        link.active ? 'bg-blue-500 text-white' : 'bg-white text-gray-700 border',
        !link.url ? 'opacity-50 cursor-not-allowed' : 'hover:bg-blue-100'
      ]"
      v-html="link.label"
    />
  </div>
  </div>

  
</template>
