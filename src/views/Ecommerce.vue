<template>
  <admin-layout>
    <div class="grid grid-cols-12 gap-4 md:gap-6">

      <div class="col-span-12 flex flex-wrap gap-4">
        <DataStatsOne />
      </div>
      
      <div class="col-span-12 xl:col-span-5">
        <customer-demographic />
      </div>
      <div class="col-span-12">
          <MeasurementsTable />
      </div>
      <div class="col-span-12">
        <all-statistics-chart />
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
import AllStatisticsChart from '../components/ecommerce/AllStatisticsChart.vue'
import RecentOrders from '../components/ecommerce/RecentOrders.vue'
import DataStatsOne from '@/components/Datasets/DataStatsOne.vue'
import MeasurementsTable from '@/components/tables/MeasurementsTable.vue'


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
    MeasurementsTable,
    AllStatisticsChart
  },
  name: 'Ecommerce',
}
</script>
