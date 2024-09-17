<template>
  <div class="container-fluid p-0">
    <!-- Komponen Navbar -->
    <NavBar :name="userName" :role="roleId" class="navbar-fixed-home"></NavBar>

    <!-- Bagian Gambar dan Teks -->
    <div class="hero-section position-relative">
      
      <img src="@/assets/home-img.jpg" class="img-fluid hero-image" alt="Gambar Latar Belakang">
      <div class="overlay-text">
        <h1 class="main-text mt-5">BOOK HERE AND GET A FREE VOUCHER</h1>
        <p class="sub-text">IDR 150.000 discount in one of our services: Spa, Scooter Rental or Food & Beverages</p>
        <p class="price-match-text">If you happen to find a better online daily rate, we will match your price.</p>
        <RouterLink class=" btn mb-5 btn-warning book-button" to="/order">ORDER NOW!</RouterLink>
        <!-- <button class="btn btn-primary book-button">BOOK NOW</button> -->
      </div>
    </div>
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
      userName: '',
      roleId: ''
    }
  },
  mounted() {
    this.userName = localStorage.getItem('name');
    this.roleId = localStorage.getItem('role_id');
    if (!this.userName || this.userName === '' || this.userName === null) {
      router.push({ name: 'login' });
    }
  },
}
</script>

<style scoped>
.navbar-fixed-home {
    position: fixed;
    top: 0;
    width: 100%;
    z-index: 1000;
}

.container-fluid {
  padding: 0;
}

.hero-section {
  height: 100vh;
  position: relative;
}

.hero-image {
  width: 100%;
  height: 100vh;
  object-fit: cover;
}

.overlay-text {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
  color: #ffffff;
  background-color: rgba(0, 0, 0, 0.5);
  border-radius:10px;
}

.main-text {
  font-size: 2.5rem;
  font-weight: bold;
  margin-bottom: 1rem;
}

.sub-text {
  font-size: 1.25rem;
  margin-bottom: 1rem;
}

.price-match-text {
  font-size: 1rem;
  margin-bottom: 1.5rem;
}

.book-button {
  background-color: #6c757d; /* Warna tombol */
  border: none;
  padding: 0.75rem 2rem;
  font-size: 1.25rem;
  border-radius: 5px;
}
</style>
