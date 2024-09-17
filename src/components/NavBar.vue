<template>
  <header v-if="$route.path !== '/login'">
    <nav class="navbar navbar-expand-lg navbar-light bg-light shadow-sm border-bottom">
      <div class="container-fluid">
        <a class="navbar-brand  text-dark" href="#" style="font-family: 'Times New Roman', Times, serif;">Food Ordering App</a>
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent"
          aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarSupportedContent">
          <ul class="navbar-nav me-auto mb-2 mb-lg-0">
            <li class="nav-item me-3">
              <RouterLink class="nav-link text-dark" to="/">Home</RouterLink>
            </li>
            <li v-if="role == 4 || role == 1" class="nav-item me-3">
              <RouterLink class="nav-link text-dark" to="/order">Order</RouterLink>
            </li>
            <li class="nav-item me-3">
              <RouterLink class="nav-link text-dark" to="/order-list">Order List</RouterLink>
            </li>
            <li v-if="role == 4" class="nav-item me-3">
              <RouterLink class="nav-link text-dark" to="/order-report">Order Report</RouterLink>
            </li>
            <li v-if="role == 4" class="nav-item me-3">
              <RouterLink class="nav-link text-dark" to="/product">Product</RouterLink>
            </li>
            <li class="nav-item">
              <a href="#" class="nav-link text-danger" @click="logout()">Logout</a>
            </li>
          </ul>
          <div class="d-flex align-items-center">
            <span class="fw-semibold text-muted">Hi, {{ name }}!</span>
          </div>
        </div>
      </div>
    </nav>
  </header>
</template>

<script>
import axios from 'axios';
import router from '@/router';

export default {
  props: ['name', 'role'],

  methods: {
    logout() {
      axios.get('http://localhost/restaurant_order_api/public/api/auth/logout', {
        headers: {
          "Authorization" : `Bearer ${localStorage.getItem('token')}`
        }
      })
      .then((response) => {
        localStorage.removeItem('email');
        localStorage.removeItem('name');
        localStorage.removeItem('role_id');
        localStorage.removeItem('token');
        router.push({ name: 'login' });
      })
      .catch((error) => {
        console.error(error);
        console.error('Terjadi kesalahan saat logout');
      });
    }
  },
}
</script>

<style scoped>
.navbar {
  background-color: #f8f9fa;
  border-bottom: 2px solid #007bff; /* Efek garis bawah */
  /* position: fixed;
    top: 0;
    width: 100%;
    z-index: 1000; */
}

.navbar-brand {
  color: #007bff;
  font-weight: bold;
  font-size: 1.5rem;
  font-family: 'Times New Roman', Times, serif; /* Font serif */
}

.nav-link {
  font-size: 1rem;
  color: #333;
  transition: color 0.3s ease;
}

.nav-link:hover {
  color: #007bff;
}

.nav-link.text-danger:hover {
  color: #ff4c4c;
}

.navbar-toggler {
  border: none;
}

.navbar-toggler-icon {
  filter: invert(1);
}

.navbar-nav .nav-item {
  margin-right: 1.5rem;
}

.fw-semibold {
  font-weight: 600;
}

.text-muted {
  color: #6c757d !important;
}

.bg-light {
  background-color: #f8f9fa !important;
}

.shadow-sm {
  box-shadow: 0 .125rem .25rem rgba(0, 0, 0, .075) !important;
}
</style>
