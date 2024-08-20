<template lang="">
   <header v-if="$route.path !== '/login'">
    <nav class="navbar navbar-expand-lg bg-body-tertiary">
      <div class="container-fluid">
        <a class="navbar-brand" href="#">Food Ordering App</a>
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent"
          aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarSupportedContent">
          <ul class="navbar-nav me-auto mb-2 mb-lg-0">
            <li class="nav-item me-3">
              <RouterLink to="/">Home</RouterLink>
            </li>
            <li class="nav-item me-3">
              <RouterLink to="/order">Order</RouterLink>
            </li>
            <li class="nav-item">
              <a href="#" @click="logout()">Logout</a>
            </li>
          </ul>
          <li class="d-flex">
           <span>Hi, {{ name }} ! </span>
          </li>
        </div>
      </div>
    </nav>
  </header>
</template>
<script>
import axios from 'axios';
import router from '@/router';


export default {
    props: ['name'],

    methods: {
    logout() {
      axios.get('http://localhost/restaurant_order_api/public/api/auth/logout', {
        headers: {
          "Authorization" : `Bearer ${localStorage.getItem('token')}`
        }
    })
      .then(function (response) {
        localStorage.removeItem('email');
        localStorage.removeItem('name');
        localStorage.removeItem('role_id');
        localStorage.removeItem('token');
        router.push({name : 'login'})
        
      })
      .catch(function (error) {
        console.log(error);
        console.log('ini error');
      });
    }
  },

}
</script>
