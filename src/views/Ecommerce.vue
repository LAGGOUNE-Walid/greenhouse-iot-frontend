<template>
  <admin-layout>
    <div class="grid grid-cols-12 gap-4 md:gap-6">

      <div class="col-span-12 flex flex-wrap gap-4">
        <DataStatsOne />
      </div>
      
      <div class="col-span-12 xl:col-span-5">
        <Suspense>

          <customer-demographic />

          <template #fallback>

            <div role="status" class="max-w-sm animate-pulse">
              <div class="h-2.5 bg-gray-200 rounded-full dark:bg-gray-700 w-48 mb-4"></div>
              <div class="h-2 bg-gray-200 rounded-full dark:bg-gray-700 max-w-[360px] mb-2.5"></div>
              <div class="h-2 bg-gray-200 rounded-full dark:bg-gray-700 mb-2.5"></div>
              <div class="h-2 bg-gray-200 rounded-full dark:bg-gray-700 max-w-[330px] mb-2.5"></div>
              <div class="h-2 bg-gray-200 rounded-full dark:bg-gray-700 max-w-[300px] mb-2.5"></div>
              <div class="h-2 bg-gray-200 rounded-full dark:bg-gray-700 max-w-[360px]"></div>
              <span class="sr-only">Loading...</span>
            </div>


          </template>
        </Suspense>
      </div>

      <div class="col-span-12">
        <statistics-chart />
      </div>
    </div>
    
  </admin-layout>
</template>

<script>

import AdminLayout from '../components/layout/AdminLayout.vue'
import EcommerceMetrics from '../components/ecommerce/EcommerceMetrics.vue'
import MonthlyTarget from '../components/ecommerce/MonthlySale.vue'
import MonthlySale from '../components/ecommerce/MonthlyTarget.vue'
import CustomerDemographic from '../components/ecommerce/CustomerDemographic.vue'
import StatisticsChart from '../components/ecommerce/StatisticsChart.vue'
import RecentOrders from '../components/ecommerce/RecentOrders.vue'
import DataStatsOne from '@/components/Datasets/DataStatsOne.vue'


export default {
  components: {
    AdminLayout,
    EcommerceMetrics,
    MonthlyTarget,
    MonthlySale,
    CustomerDemographic,
    StatisticsChart,
    RecentOrders,
    DataStatsOne,
  },
  name: 'Ecommerce',
}
</script>
