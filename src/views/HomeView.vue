<template lang="">
  <!-- navbar component disini -->
  <NavBar :name=userName />
  <div>
    {{userName}}
  </div>
</template>
<script>
import router from '@/router';
import NavBar from '@/components/NavBar.vue';

export default {
  components: {
    NavBar
  },
  data() {
    return {
      userName: ""
    }
  },
  mounted() {
    this.userName = localStorage.getItem('name')
    if (!this.userName || this.userName == '' || this.userName == null) {
      router.push({name : 'login'})
    }
  },
}
</script>
<style lang="">
  
</style>