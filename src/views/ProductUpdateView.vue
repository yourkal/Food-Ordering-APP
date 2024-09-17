<template>
    <!-- navbar component disini -->
    <NavBar :name="userName" :role="roleId" />
    <div class="container3 mt-5 p-4 shadow-sm rounded bg-light">
      <h2 class="text-center mb-4">Edit Product</h2>
      <hr/>
      <form @submit.prevent="updateProduct()">
        <div class="form-group">
          <label for="name">Product Name</label>
          <input type="text" id="name" v-model="item.name" class="form-control" />
        </div>
        <div class="form-group">
          <label for="price">Product Price</label>
          <input type="number" id="price" v-model="item.price" class="form-control" />
        </div>
        <div class="form-group">
          <label for="image">Current Image</label> <br>
          <img v-if="item.image" :src="url+item.image" class="product-image" />
          <img v-else src="@/assets/images/default-image.jpg" class="product-image" />
        </div>
        <div class="form-group">
          <label for="image">Product Image</label>
          <input type="file" id="image" @change="imageChanged($event)" class="form-control" />
          <img :src="image" class="product-image mt-3" v-if="image"/>
        </div>
        <button type="submit" class="btn3 btn-success"><i class="fas fa-save"></i>Update Product</button>
      </form>
    </div>
  </template>
  
  <script>
  import NavBar from '@/components/NavBar.vue';
  import axios from 'axios';
  import router from '@/router';
  
  export default {
    components: {
        NavBar
    },
    data() {      
      return {
        userName: "",
        roleId: "",
        url: 'http://localhost/restaurant_order_api/public/storage/items/',
        productId: '',
        item: '',
        file: '',       
        // name: '',
        // price: '',
        // image: '',
      };
    },

    mounted() {
        this.productId = this.$route.params.productId
        this.userName = localStorage.getItem('name')
        this.roleId = localStorage.getItem('role_id')

        if (!this.userName || this.userName === '' || this.userName === null) {
            router.push({ name: 'login' });
        }
        if (this.roleId != 4) {
            router.push({ name: 'home' });
        }

        this.showItem();
    },

    methods: {
        showItem() {
            axios.get('http://localhost/restaurant_order_api/public/api/item/'+this.productId + '', {
                headers: {
                    "Authorization": `Bearer ${localStorage.getItem('token')}`
                }
            })
                .then((response) => {
                    this.item = response.data.data;
                    // console.log(response.data.data);
                })
                .catch((error) => {
                    console.log(error);
                    console.log('Error get item');
                });
        },

        updateProduct() {
          if (!this.item.name || !this.item.price) {
            Swal.fire({
            icon: "error",
            title: "Oops...",
            text: "Product Name & Price is Required!",
            // footer: '<a href="#">Why do I have this issue?</a>'
          });
            return;
          }

        let formData = new FormData();
          formData.append('name', this.item.name);
          formData.append('price', this.item.price);
          formData.append('image_file', this.file);
          formData.append('_method', 'patch');

          axios.post('http://localhost/restaurant_order_api/public/api/item/'+this.productId, formData, {
        headers: {
            // 'Content-Type': 'multipart/form-data',
            "Authorization": `Bearer ${localStorage.getItem('token')}`,
          }
        })
        .then((response) => {
          router.push({ name: 'product' });
          // this.items = response.data.data;
        })
          .catch((error) => {
          console.log(error);
          console.log('Error get item');
        });
      },

      imageChanged(e){
        let file = e.target.files[0];
        this.file = file;
      },
    }
  }

  </script>
  
  <style>
  .container3 {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}
  .product-image {
    width: 100px;
    height: 100px;
    border-radius: 10px;
  }

  .form-group {
  margin-bottom: 20px; /* Margin bawah yang lebih besar */
}

.form-control {
  padding: 10px;
  border-radius: 5px;
}
.btn:hover {
  background-color: #0056b3; /* Warna hover untuk tombol */
}

h2 {
  font-weight: bold;
  color: #343a40;
}

label {
  font-weight: 500;
}

input::placeholder {
  font-style: italic;
}

.btn3 {
  background-color: #007bff;
  border: none;
  color: white;
  padding: 10px 20px;
  font-size: 16px;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}
  </style>
  