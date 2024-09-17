<template lang="">
    <!-- navbar component disini -->
    <NavBar :name="userName" :role="roleId" />
    <div class="container-order-detail mt-5">
        <h1 class="text-center mb-4">Order List</h1>

        <!-- Search Input -->
        <div class="row mb-3">
            <div class="col-12 md-4">
                <input 
                    type="text" 
                    class="form-control" 
                    placeholder="Search orders..." 
                    v-model="searchQuery"
                />
            </div>
        </div>

        <table class="table table-striped table-hover">
            <thead>
                <tr>
                    <th>No</th>
                    <th>Customer Name</th>
                    <th>Table No</th>
                    <th>Order Date</th>
                    <th>Order Time</th>
                    <th>Status</th>
                    <th>Total</th>
                    <th>Waitress</th>
                    <th>Cashier</th>
                    <th>Action</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="(order, index) in filteredOrders" :key="index">
                    <td>{{ (currentPage - 1) * itemsPerPage + index + 1 }}</td>
                    <td>{{ order.customer_name }}</td>
                    <td>{{ order.table_no }}</td>
                    <td>{{ order.order_date }}</td>
                    <td>{{ order.order_time }}</td>
                    <td>{{ order.status }}</td>
                    <td>{{ 'Rp ' + order.total }}</td>
                    <td>{{ order.waitress ? order.waitress.name : '-'  }}</td>
                    <td>{{ order.cashier ? order.cashier.name : '' }}</td>
                    <td>
                        <RouterLink :to="{ name: 'orderDetail', params: { orderId: order.id }}">
                                <button class="btn btn-sm btn-warning mr-2">
                                    <i class="fas fa-eye"></i>
                                </button>
                            </RouterLink>
                            <!-- <RouterLink to="/delete-page">
                                <button class="btn btn-sm btn-danger">
                                    <i class="fas fa-trash-alt"></i>
                                </button>
                            </RouterLink> -->
                        <button class="btn btn-sm btn-danger"  @click="deleteOrder(order.id)" >
                            <i class="fas fa-trash-alt"></i>
                        </button>
                    </td>
                </tr>
            </tbody>
        </table>

        <!-- Pagination Controls -->
        <nav aria-label="Page navigation example">
            <ul class="pagination justify-content-center">
                <li class="page-item" :class="{ disabled: currentPage === 1 }">
                    <button class="page-link" @click="prevPage">Previous</button>
                </li>
                <li class="page-item" :class="{ active: n === currentPage }" v-for="n in totalPages" :key="n">
                    <button class="page-link" @click="goToPage(n)">{{ n }}</button>
                </li>
                <li class="page-item" :class="{ disabled: currentPage === totalPages }">
                    <button class="page-link" @click="nextPage">Next</button>
                </li>
            </ul>
        </nav>
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
            orders: [],  // Array untuk menyimpan daftar pesanan
            currentPage: 1,
            itemsPerPage: 5,  // Jumlah item per halaman
            searchQuery: ""  // Query untuk pencarian
        }
    },

    mounted() {
        this.userName = localStorage.getItem('name');
        this.roleId = localStorage.getItem('role_id');

        if (!this.userName || this.userName === '' || this.userName === null) {
            router.push({ name: 'login' });
        }

        this.getOrders();  // Panggil metode untuk mengambil daftar pesanan
    },

    computed: {
        totalPages() {
            return Math.ceil(this.filteredOrders.length / this.itemsPerPage);
        },

        filteredOrders() {
            return this.orders.filter(order => {
                const searchLower = this.searchQuery.toLowerCase();
                return order.customer_name.toLowerCase().includes(searchLower) ||
                    order.table_no.toString().includes(searchLower) ||
                    order.waitress.name.toLowerCase().includes(searchLower) ||
                    (order.cashier ? order.cashier.name.toLowerCase() : '').includes(searchLower);
            });
        },

        paginatedOrders() {
            const start = (this.currentPage - 1) * this.itemsPerPage;
            const end = start + this.itemsPerPage;
            return this.filteredOrders.slice(start, end);
        }
    },

    methods: {
        getOrders() {
            axios.get('http://localhost/restaurant_order_api/public/api/order', {
                headers: {
                    "Authorization": `Bearer ${localStorage.getItem('token')}`
                }
            })
            .then((response) => {
                console.log(response.data.data); // Debug: memastikan data diterima
                this.orders = response.data.data;
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
        },

        deleteOrder(orderId) {
            Swal.fire({
                title: 'Are you sure?',
                text: "You won't be able to revert this!",
                icon: 'warning',
                showCancelButton: true,
                confirmButtonText: 'Yes, delete it!',
                cancelButtonText: 'No, cancel!',
            }).then((result) => {
                if (result.isConfirmed) {
                    axios.delete(`http://localhost/restaurant_order_api/public/api/order/${orderId}`, {
                        headers: {
                            "Authorization": `Bearer ${localStorage.getItem('token')}`
                        }
                    })
                    .then(() => {
                        Swal.fire('Deleted!', 'Order has been deleted.', 'success');
                        this.getOrders();  // Refresh daftar pesanan setelah penghapusan
                    })
                    .catch((error) => {
                        console.error('Error deleting order:', error);
                        Swal.fire('Error', 'Failed to delete order. Please try again later.', 'error');
                    });
                }
            });
        },

        goToPage(pageNumber) {
            this.currentPage = pageNumber;
        },

        prevPage() {
            if (this.currentPage > 1) {
                this.currentPage--;
            }
        },

        nextPage() {
            if (this.currentPage < this.totalPages) {
                this.currentPage++;
            }
        }
    }
}
</script>



<style>
.container-order-detail {
    max-width: 1500px;
    margin-left: 10px;
    margin-right: 20px;
}

.table th, .table td {
    vertical-align: middle;
}

.table-hover tbody tr:hover {
    background-color: #f1f1f1;
}

.btn {
    margin-right: 5px;
}
</style>
