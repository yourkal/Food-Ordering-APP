<template>
    <div class="fixed-content">
        <!-- navbar component disini -->
        <NavBar :name="userName" :role="roleId" class="navbar-fixed" />
        
        <div class="container-fluid"> 
            <div class="row">
                <!-- item list -->
                <div class="col-12 col-sm-8 mb-3">
                   <!-- search box -->
                   <div class="col-12 mb-3 ">
                        <input type="text" v-model="keyword" class="form-control" placeholder="Cari Menu..." :onchange="searchItem()" />
                        <!-- <button class="btn btn-warning" type="submit">Cari</button> -->
                   </div>
                   <hr/>
                   <div class="col-12">
                        <div class="item-list-container">
                            <div class="row">
                                <div v-for="item in filteredItems" :key="item.id" class="col-12 col-sm-6 col-md-4 col-lg-3 mb-3">
                                    <div class="card">
                                        <img :src="url+item.image" class="card-img-top object-fit-cover" height="200" :alt="item.name">
                                        <div class="card-body text-center">
                                            <h5 class="card-title text-center">{{ item.name }}</h5>
                                            <p class="card-text">{{ 'Rp ' + item.price }}</p>
                                            <button class="btn btn-primary" @click="orderItem(item.id)">Order</button>
                                        </div>                                
                                    </div>
                                </div>
                            </div>
                        </div>
                   </div>
                </div>

               <!-- ordered item -->
               <div class="col-12 col-sm-4 mb-3 ordered-box">
                    <h4 class="mb-4">Ordered List</h4>
                    <div class="order-list order-box">
                        <div class="mb-4">
                        <div>
                            <label for="customerName" class="form-label">Customer Name</label>
                            <input type="text" class="form-control" id="customerName" v-model="customerName">
                        </div>
                        <div class="mt-2">
                            <label for="tableNo" class="form-label">Table No</label>
                            <input type="text" class="form-control" id="tableNo" v-model="tableNo">
                        </div>
                    </div>
                        <!-- Daftar item yang dipesan -->
                        <ul  v-for="order in orders" class="list-group">
                            <li class="list-group-item d-flex justify-content-between align-items-center">
                                <div class="d-flex align-items-center">
                                    <span class="badge bg-primary rounded-pill me-2">{{ order.qty }}</span>
                                    <div>
                                        <h6 class="mb-0">{{order.name}}</h6>
                                        <small>{{ 'Rp ' + order.eachPrice }}</small>
                                    </div>
                                </div>
                                <div>
                                    <div><strong class="ms-auto">{{ 'Rp ' + order.price }}</strong></div>
                                    <div class="d-flex justify-content-between align-items-center">
                                        <div class="button-group d-flex">                                        
                                            <button class="btn btn-outline-primary btn-sm" @click="decrementItemQty(order)">−</button>
                                            <button class="btn btn-outline-primary btn-sm" @click="incrementItemQty(order)">+</button>
                                            <!-- <button class="btn btn-outline-primary btn-sm " @click="incrementItemQty(order)">+</button> -->
                                            <button class="btn btn-outline-danger btn-sm" @click="deleteItem">
                                                <i class="fas fa-trash-alt"></i>
                                            </button>
                                        </div>
                                    </div>
                                </div>
                            </li>                           
                        </ul>
                    </div>
                    
                    <div class="order-summary  ">
                        <div class="total">
                            <strong>Total:</strong>
                            <strong>{{ total }}</strong>
                        </div>
                        <button @click="submitOrder()" class="btn btn-success w-100 mt-3 submit-btn" 
                        :disabled="process">{{process ? 'Processing Order...' : 'Submit'}}</button>
                    </div>
                </div>

            </div>
        </div>
    </div>
</template>

<script>
import Swal from 'sweetalert2';
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
            url: 'http://localhost/restaurant_order_api/public/storage/items/',
            orders: [],
            total: 0,
            customerName: '',
            tableNo: '',
            process: false
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
                if(error.response.status == 401){
                        localStorage.removeItem('token');
                        localStorage.removeItem('email');
                        localStorage.removeItem('name');
                        localStorage.removeItem('role_id');

                        router.push({ name: 'login' });
                        
                    }
                    console.log(error);
                    console.log('Error get item');
            });
        },
        searchItem() {
           this.filteredItems = this.items.filter(item => item.name.toLowerCase().includes(this.keyword.toLowerCase()));
        },

        orderItem(id){
            // console.log(id)
            let item = this.filteredItems.filter(item => item.id == id)[0]
            // console.log(item)
            let orderItem = Object.assign({}, item)
            orderItem.eachPrice = item.price
            let orderQty = this.orders.map(e => e.id).indexOf(orderItem.id)

            if(orderQty != -1){ //jika item yang sudah ada di orders, maka maka qty ditambahkan +1 & mengjumlahkan qty
                this.orders[orderQty].qty ++
                // eachPrice ditambahkan dengan eachPrice sebelumnya
                // eachPerice = 20k
                //20k * 1 = price 20k, eachPrice = 20k
                //20k * 2 = price 40k, eachPrice = 20k
                //20k * 3 = price 60k, eachPrice = 20k
                this.orders[orderQty].price = this.orders[orderQty].qty * this.orders[orderQty].eachPrice
            }
            else{ //jika item yang dipilih belum ada di orders, maka ditambahkan ke orders hanya 1
                orderItem.qty = 1
                this.orders.push(orderItem)
            }

            let orderPrice = this.orders.map(order => order.price)
            let priceTotal = 0

            orderPrice.forEach(price => {
                priceTotal += price
            })

            this.total = priceTotal

        },

        decrementItemQty(item){
            //hitung qty (done)
            //hitung price (done)
            //hitung total
            
            //qty gaboleh kurang dari 1
            if(item.qty <= 1) {
                return
            }

            let orderQty = this.orders.map(e => e.id).indexOf(item.id)
            this.orders[orderQty].qty --
            this.orders[orderQty].price = this.orders[orderQty].qty * this.orders[orderQty].eachPrice

            let orderPrice = this.orders.map(order => order.price)
            let priceTotal = 0

            orderPrice.forEach(price => {
                priceTotal += price
            })
            this.total = priceTotal
            console.log(orderPrice)
        },
        incrementItemQty(item){
            //hitung qty (done)
            //hitung price (done)
            //hitung total
    
            let orderQty = this.orders.map(e => e.id).indexOf(item.id)
            this.orders[orderQty].qty ++
            this.orders[orderQty].price = this.orders[orderQty].qty * this.orders[orderQty].eachPrice
    
            let orderPrice = this.orders.map(order => order.price)
            let priceTotal = 0
    
            orderPrice.forEach(price => {
                priceTotal += price
            })
            this.total = priceTotal
            console.log(orderPrice)
        },

        deleteItem(item){
            let orderQty = this.orders.map(e => e.id).indexOf(item.id)
            this.orders.splice(orderQty, 1)

            let orderPrice = this.orders.map(order => order.price)
            let priceTotal = 0
            orderPrice.forEach(price => {
                priceTotal += price
            })
            this.total = priceTotal
        },

        submitOrder(){
            if(this.customerName == '' || this.tableNo == ''){
                Swal.fire({
                icon: "error",
                title: "Oops...",
                text: "Customer Name & Table No is Required",
                // footer: '<a href="#">Why do I have this issue?</a>'
                });
            }
            let item = this.orders.map(item => {
                return{
                    'id' : item.id,
                    'qty' : item.qty
                }
            })
            // let itemId = this.orders.map(item => item.id)
            
            this.process = true //menonaktifkan button submit
            axios.post('http://localhost/restaurant_order_api/public/api/order', {
                'customer_name' : this.customerName,
                'table_no' : this.tableNo,
                'items' : item
            }, {
                headers: {
                    "Authorization" : `Bearer ${localStorage.getItem('token')}`
                }
            })
            .then((response) => {
                // console.log(response.data.data); // Debug: memastikan data diterima
               console.log(response)
                Swal.fire({
                icon: "success",
                title: "Order Succes Submit",
                text: "click ok to continue",
                // footer: '<a href="#">Why do I have this issue?</a>'
                })

                //menghapus semua item di orders ketika order disubmit
                this.orders = []
                this.total = 0
                this.customerName = ''
                this.tableNo = ''

            })
            .catch((error) => {
                if(error.response.status == 401){
                        localStorage.removeItem('token');
                        localStorage.removeItem('email');
                        localStorage.removeItem('name');
                        localStorage.removeItem('role_id');

                        router.push({ name: 'login' });
                        
                    }
                    console.log(error);
                    console.log('Error get item');
            })
            .finally(() => this.process = false); //mengaktifkan button submit
            // console.log(itemId)
        }
    },

}
</script>

<style>
.orered-box{
    border-left: 1px solid #dee2e6 !important;
}

.navbar-fixed {
    position: fixed;
    top: 0;
    width: 100%;
    z-index: 1000;
}

.fixed-content {
    padding-top: 80px; /* Sesuaikan dengan tinggi navbar */
    top: 0;
    bottom: 0;
    position: fixed;
    overflow-y: scroll;
    overflow-x: hidden;
}

.order-box{
    border-left: 1px solid #dee2e6 !important;
    max-height: 400px; /* Sesuaikan tinggi maksimal sesuai kebutuhan */
    overflow-y: auto; /* Aktifkan scrollbar vertikal jika konten melebihi tinggi maksimal */
    
}

.order-box::-webkit-scrollbar {
    display: none; /* Sembunyikan scrollbar */
}

.order-box h4 {
    border-bottom: 2px solid #dee2e6;
    padding-bottom: 10px;
}

.order-box .list-group-item {
    border: none;
    border-bottom: 1px solid #dee2e6;
}

.order-box .badge {
    font-size: 1rem;
    padding: 10px;
}

.order-box .btn-danger {
    padding: 5px 10px;
}

.button-group {
    display: flex;
    align-items: center;
}

.button-group button {
    min-width: 20px;
    border-radius: 4px;
    font-size: 15px;
    transition: background-color 0.3s ease, color 0.3s ease;
}

.button-group button.btn-outline-primary {
    border-color: #007bff;
    color: #007bff;
}

.button-group button.btn-outline-primary:hover {
    background-color: #007bff;
    color: white;
}

.button-group button.btn-outline-danger {
    border-color: #dc3545;
    color: #dc3545;
}

.button-group button.btn-outline-danger:hover {
    background-color: #dc3545;
    color: white;
}

/* New styles for item list scrollbar */
.item-list-container {
    max-height: 450px; /* Sesuaikan sesuai kebutuhan */
    overflow-y: scroll; /* Pastikan menggulir diaktifkan */
    padding-right: 5px; /* Ruang untuk scrollbar, sesuaikan jika perlu */
}

/* Sembunyikan scrollbar tetapi tetap memungkinkan menggulir */
.item-list-container {
    scrollbar-width: none; /* Untuk Firefox */
    -ms-overflow-style: none; /* Untuk Internet Explorer dan Edge */
}

/* Sembunyikan scrollbar untuk browser WebKit */
.item-list-container::-webkit-scrollbar {
    display: none; /* Sembunyikan scrollbar */
}

/* Styling untuk total dan submit order */
.order-summary {
    border-top: 1px solid #dee6df;
    padding: 5px;
    background-color: #fff;
    
    
}

.order-summary .total {
    display: flex;
    justify-content: space-between;
}

.order-summary .submit-btn {
    margin-top: 10px;
}

</style>
