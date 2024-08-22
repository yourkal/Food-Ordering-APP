<template>
    <div>
        <!-- navbar component disini -->
        <NavBar :name="userName" :role="roleId" />
        
        <div class="container-fluid mt-5"> 
            <div class="row">
                <!-- item list -->
                <div class="col-12 col-sm-8 mb-3">
                   <!-- search box -->
                   <div class="col-12 mb-3">
                        <input type="text" v-model="keyword" class="form-control" placeholder="Cari Menu..." :onchange="searchItem()" />
                        <!-- <button class="btn btn-warning" type="submit">Cari</button> -->
                   </div>
                   <hr/>
                   <div class="col-12">
                        <div class="row">
                            <div v-for="item in filteredItems" :key="item.id" class="col-12 col-sm-6 col-md-4 col-lg-3 mb-3">
                                <div class="card">
                                    <img :src="url+item.image" class="card-img-top object-fit-cover" height="200" :alt="url">
                                    <div class="card-body text-center">
                                        <h5 class="card-title text-center">{{ item.name }}</h5>
                                        <p class="card-text">{{ 'Rp ' + item.price }}</p>
                                        <button class="btn btn-primary">Order</button>
                                    </div>                                
                                </div>
                            </div>
                        </div>
                   </div>
                </div>

                <!-- ordered item -->
                <div class="col-12 col-sm-4 mb-3 bordered">
                    jciudsciu
                </div>
            </div>
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
            filteredItems: [],
            keyword: '',
            url: 'http://localhost/restaurant_order_api/public/storage/items/'
        }
    },
    mounted() {
        this.userName = localStorage.getItem('name')
        this.roleId = localStorage.getItem('role_id')
        if (!this.userName || this.userName === '' || this.userName === null) {
            router.push({ name: 'login' });
        }
        if (this.roleId != 4 && this.roleId != 1) {
            router.push({ name: 'home' });
        }
        this.getItems();
    },
    methods: {
        getItems() {
            axios.get('http://localhost/restaurant_order_api/public/api/item', {
                headers: {
                    "Authorization" : `Bearer ${localStorage.getItem('token')}`
                }
            })
            .then((response) => {
                // console.log(response.data.data); // Debug: memastikan data diterima
                this.items = response.data.data;
            })
            .catch((error) => {
                console.log(error)
                console.log('Error get item')
            });
        },
        searchItem() {
           this.filteredItems = this.items.filter(item => item.name.toLowerCase().includes(this.keyword.toLowerCase()));
        }
    }
}
</script>

<style>
.bordered {
    border: solid 1px;
}
</style>
