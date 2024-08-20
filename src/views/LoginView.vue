<template>
  <section class="vh-100 d-flex flex-column">
    <div class="container-fluid h-custom flex-grow-1">
      <div class="row d-flex justify-content-center align-items-center h-100">
        <div class="col-md-6">
          <img src="@/assets/bg-login2.png" class="img-fluid" alt="Sample image">
        </div>
        <div class="col-md-4"> <!-- Mengubah ukuran kolom menjadi col-md-4 -->
          <form @submit.prevent class="login-form p-4">
            <!-- Email input -->
            <div class="form-outline mb-4">
              <h1 class="text-center mb-4">Login</h1>
              <hr>
              <label class="form-label" for="email">Email address</label>
              <input type="email" v-model="email" id="email" class="form-control form-control-lg"
                placeholder="Enter a valid email address" required />
            </div>

            <!-- Password input -->
            <div class="form-outline mb-3">
              <label class="form-label" for="password">Password</label>
              <input type="password" v-model="password" id="password" class="form-control form-control-lg"
                placeholder="Enter password" required />
            </div>

            <button @click="login" class="btn btn-primary btn-lg" style="padding-left: 2.5rem; padding-right: 2.5rem;">
              Login
            </button>
          </form>
        </div>
      </div>
    </div>
    <!-- <div class="footer d-flex justify-content-center py-3 bg-primary">
      <div class="text-white">Copyright © 2024. All rights reserved.</div>
    </div> -->
  </section>
</template>

<script>
import axios from 'axios';
import router from '@/router';

export default {
  data() {
    return {
      email: "",
      password: ""
    };
  },
  methods: {
    login() {
      axios.post('http://localhost/restaurant_order_api/public/api/auth/login', {
        email: this.email,
        password: this.password
      })
        .then(function (response) {
          localStorage.setItem('email', response.data.data.email);
          localStorage.setItem('name', response.data.data.name);
          localStorage.setItem('role_id', response.data.data.role_id);
          localStorage.setItem('token', response.data.data.token);
          router.push({name : 'home'})
          
        })
        .catch(function (error) {
          console.log(error);
          console.log('ini error');
        });
    }
  },
};
</script>

<style>
/* CSS for smaller and centered login form */
.login-form {
  border: 1px solid #ccc; /* Garis tepi abu-abu */
  border-radius: 8px; /* Membuat sudut tepi melengkung */
  background-color: #fff; /* Latar belakang putih */
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1); /* Menambahkan bayangan halus */
}
</style>
