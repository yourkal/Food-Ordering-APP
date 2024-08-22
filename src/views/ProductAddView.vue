<template lang="">
    <!-- navbar component disini -->
    <NavBar :name="userName" :role="roleId" />
    <div class="container2 mt-5">
    <h2 class="text-center mb-4">Add Product</h2>
    <div class="form-container p-4 shadow-sm rounded bg-light">
      <form @submit.prevent="createProduct()">
        <div class="form-group">
          <label for="name">Product Name</label>
          <input type="text" id="name" v-model="name" class="form-control" placeholder="Enter product name"  />
        </div>
        <div class="form-group">
          <label for="price">Price</label>
          <input type="number" id="price" v-model="price" class="form-control" placeholder="Enter product price"  />
        </div>
        <div class="form-group">
           <label for="image">Image</label>
           <td >
           <input type="file" id="image" @change="imageChanged($event)" class="form-control" />
           </td>
        </div>
    
        <button type="submit" class="btn2 btn-primary btn-block mt-4 form-control">
          <i class="fas fa-save"></i> Save
        </button>
        
      </form>
    </div>
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
            items: [],
            url: 'http://localhost/restaurant_order_api/public/storage/items/',
            name: '',
            price: '',
            file: ''
            
           
        }
    },

    methods: {
    createProduct() {

      if (!this.name || !this.price) {
        alert('Please fill in all fields');
        return;
      }

      // let data = {
      //   name: this.name,
      //   price: this.price
      // }

      let formData = new FormData();
      formData.append('name', this.name);
      formData.append('price', this.price);
      formData.append('image_file', this.file);

      axios.post('http://localhost/restaurant_order_api/public/api/item', formData, {
        headers: {
            'Content-Type': 'multipart/form-data',
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

  },
    mounted() {
        this.userName = localStorage.getItem('name')
        this.roleId = localStorage.getItem('role_id')

        if (!this.userName || this.userName === '' || this.userName === null) {
            router.push({ name: 'login' });
        }
        if (this.roleId != 4) {
            router.push({ name: 'home' });
        }
    },
    
}
</script>

 <style >
.container2 {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}


.form-group {
  margin-bottom: 20px; /* Margin bawah yang lebih besar */
}

.form-control {
  padding: 10px;
  border-radius: 5px;
}

.btn2 {
  background-color: #007bff;
  border: none;
  color: white;
  padding: 10px 20px;
  font-size: 16px;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease;
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
</style>