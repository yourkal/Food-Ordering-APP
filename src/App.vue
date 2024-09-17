<script>
import { RouterLink, RouterView } from 'vue-router'
import router from '@/router';

export default {
  // mounted() {
  //   this.userName = localStorage.getItem('name')
  //   if (this.userName || this.userName != '' || this.userName != null) {
  //     router.push({ name: 'home' })
  //   }
  // },
}
</script>

<template>
  <RouterView />
</template>