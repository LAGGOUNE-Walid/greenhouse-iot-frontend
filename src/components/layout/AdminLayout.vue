<template>
  <div class="min-h-screen xl:flex" v-if="signed">
    <Backdrop />
    <div
      class="flex-1 transition-all duration-300 ease-in-out"
    >
      <app-header />
      <div class="p-4 mx-auto max-w-screen-2xl md:p-6">
        <slot></slot>
      </div>
    </div>
  </div>
</template>

<script setup>
import AppSidebar from './AppSidebar.vue'
import AppHeader from './AppHeader.vue'
import { useSidebar } from '@/composables/useSidebar'
import Backdrop from './Backdrop.vue'
const { isExpanded, isHovered } = useSidebar()
import { useRouter } from 'vue-router';
import { onMounted, reactive, ref } from 'vue';
const userData = reactive({
    user: {}
});
var signed = ref(false)
function fetchUserDetails() {
    axios.get('api/user').then(response => {
        userData.user = response.data;
        if (Object.keys(userData.user).length > 0) {
          signed.value = true;
        }
    });
}
function signOut() {
    axios.post('api/logout').finally(() => {
        localStorage.removeItem('token');
        axios.defaults.headers.common['Authorization'] = 'Bearer';
        router.push({ name: 'Signin' });
    });
}
onMounted(fetchUserDetails);
</script>
