<template>
    <div class="container mt-5">
      <h2 class="text-center mb-4">Edit Product</h2>
      <hr/>
      <form @submit.prevent="updateProduct">
        <div class="form-group">
          <label for="name">Product Name</label>
          <input type="text" id="name" v-model="product.name" class="form-control" required/>
        </div>
        <div class="form-group">
          <label for="price">Product Price</label>
          <input type="number" id="price" v-model="product.price" class="form-control" required/>
        </div>
        <div class="form-group">
          <label for="image">Product Image</label>
          <input type="file" id="image" @change="onFileChange" class="form-control" />
          <img :src="product.image_url" class="product-image mt-3" v-if="product.image_url"/>
        </div>
        <button type="submit" class="btn btn-success">Update Product</button>
      </form>
    </div>
  </template>
  
  <script>
  import axios from 'axios';
  import router from '@/router';
  
  export default {
    data() {
      return {
        product: {
          name: '',
          price: '',
          image: '',
          image_url: ''
        },
        selectedFile: null
      };
    },
    props: ['id'],
    mounted() {
      this.getProduct();
    },
    methods: {
      getProduct() {
        axios.get(`http://localhost/restaurant_order_api/public/api/item/${this.id}`, {
          headers: {
            "Authorization" : `Bearer ${localStorage.getItem('token')}`
          }
        })
        .then((response) => {
          this.product = response.data.data;
          this.product.image_url = `http://localhost/restaurant_order_api/public/storage/items/${this.product.image}`;
        })
        .catch((error) => {
          console.log(error);
        });
      },
      onFileChange(e) {
        this.selectedFile = e.target.files[0];
      },
      updateProduct() {
        let formData = new FormData();
        formData.append('name', this.product.name);
        formData.append('price', this.product.price);
        if (this.selectedFile) {
          formData.append('image', this.selectedFile);
        }
  
        axios.post(`http://localhost/restaurant_order_api/public/api/item/${this.id}/update`, formData, {
          headers: {
            'Content-Type': 'multipart/form-data',
            "Authorization" : `Bearer ${localStorage.getItem('token')}`
          }
        })
        .then((response) => {
          router.push({ name: 'home' });
        })
        .catch((error) => {
          console.log(error);
        });
      }
    }
  };
  </script>
  
  <style>
  .product-image {
    width: 100px;
    height: 100px;
    border-radius: 10px;
  }
  </style>
  