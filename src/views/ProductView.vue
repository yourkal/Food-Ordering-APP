<template lang="">
    <!-- navbar component disini -->
    <NavBar :name="userName" :role="roleId" />
    <div>
        <div class="container mt-3">
            <h2 class="text-center mb-3">Product List</h2>
            <RouterLink :to="{ name: 'productAdd' }" class="btn btn-primary">
                <i class="fas fa-plus"></i> Add Product
            </RouterLink>

            <table class="table table-striped table-hover">
                <thead>
                    <tr>
                        <th>No</th>
                        <th>Name</th>
                        <th>Price</th>
                        <th>Image</th>
                        <th>Action</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(item, index) in paginatedItems" :key="index">
                        <td>{{ (currentPage - 1) * itemsPerPage + index + 1 }}</td>
                        <td>{{item.name}}</td>
                        <td>{{'Rp '+item.price}}</td>
                        <td>
                            <img v-if="item.image" :src="url+item.image" class="product-image" />
                            <img v-else src="@/assets/images/default-image.jpg" class="product-image" />
                        </td>
                        <td>
                            <RouterLink :to="{ name: 'productEdit', params: { productId: item.id }}">
                                <button class="btn btn-sm btn-warning mr-2">
                                    <i class="fas fa-edit"></i>
                                </button>
                            </RouterLink>
                            <button class="btn btn-sm btn-danger" @click="deleteItem(item.id)">
                                <i class="fas fa-trash-alt"></i>
                            </button>
                        </td>
                    </tr>
                </tbody>
            </table>

            <!-- Pagination Controls -->
            <nav aria-label="Page navigation">
                <ul class="pagination justify-content-center">
                    <li class="page-item" :class="{ disabled: currentPage === 1 }">
                        <a class="page-link" href="#" @click.prevent="prevPage">Previous</a>
                    </li>
                    <li class="page-item" :class="{ disabled: currentPage === totalPages }">
                        <a class="page-link" href="#" @click.prevent="nextPage">Next</a>
                    </li>
                </ul>
            </nav>
        </div>
    </div>
</template>

<script>
import NavBar from '@/components/NavBar.vue';
import axios from 'axios';
import Swal from 'sweetalert2';
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
            currentPage: 1,
            itemsPerPage: 5
        }
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

        this.getItems();
    },

    computed: {
        paginatedItems() {
            const start = (this.currentPage - 1) * this.itemsPerPage;
            const end = start + this.itemsPerPage;
            return this.items.slice(start, end);
        },
        totalPages() {
            return Math.ceil(this.items.length / this.itemsPerPage);
        }
    },

    methods: {
        getItems() {
            axios.get('http://localhost/restaurant_order_api/public/api/item', {
                headers: {
                    "Authorization": `Bearer ${localStorage.getItem('token')}`
                }
            })
                .then((response) => {
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
        deleteItem(itemId) {
    // Konfirmasi penghapusan dengan SweetAlert
    Swal.fire({
        title: 'Are you sure?',
        text: "Do you want to delete this product?",
        icon: 'warning',
        showCancelButton: true,
        confirmButtonColor: '#3085d6',
        cancelButtonColor: '#d33',
        confirmButtonText: 'Yes, delete it!'
    }).then((result) => {
        if (result.isConfirmed) {
            // Kirim permintaan penghapusan ke server
            axios.delete(`http://localhost/restaurant_order_api/public/api/item/${itemId}`, {
                headers: {
                    "Authorization": `Bearer ${localStorage.getItem('token')}`
                }
            })
            .then((response) => {
                if (response.status === 200 || response.status === 204) {
                    // Jika penghapusan berhasil, perbarui daftar item
                    this.items = this.items.filter(item => item.id !== itemId);
                    Swal.fire(
                        'Deleted!',
                        'The product has been deleted.',
                        'success'
                    );
                } else {
                    Swal.fire(
                        'Error!',
                        'Failed to delete the product. Please try again.',
                        'error'
                    );
                }
            })
            .catch((error) => {
                console.log(error);
                Swal.fire(
                    'Error!',
                    'Failed to delete the product.',
                    'error'
                );
            });
        }
    });
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
        },
    }
}
</script>


<style>
.table {
    margin-top: 20px;
    border-collapse: separate;
    border-spacing: 0 10px;
    border: none;
}

.table th,
.table td {
    border-top: none !important;
    background: #f8f9fa;
    text-align: center;
    vertical-align: middle;
}

.table-striped tbody tr:nth-of-type(odd) {
    background-color: #e9ecef;
}

.table-hover tbody tr:hover {
    background-color: #dee2e6;
}

.table thead th {
    border-bottom: none;
    font-weight: bold;
    background-color: #007bff;
    color: white;
    text-transform: uppercase;
}

.table thead th:first-child {
    border-top-left-radius: 10px;
}

.table thead th:last-child {
    border-top-right-radius: 10px;
}

.table tbody tr {
    border-radius: 10px;
}

.table tbody td {
    border: none;
}

.table img {
    width: 50px;
    height: 50px;
    border-radius: 10px;
}

.btn {
    margin: 0 5px;
    border-radius: 5px;
}

.btn-warning {
    background-color: #ffc107;
    border: none;
    color: #fff;
}

.btn-danger {
    background-color: #dc3545;
    border: none;
    color: #fff;
}

.pagination {
    margin-top: 20px;
}

.page-link {
    color: #007bff;
}

.page-item.disabled .page-link {
    color: #6c757d;
    pointer-events: none;
    background-color: #fff;
    border-color: #dee2e6;
}

.page-link:hover {
    color: #0056b3;
    background-color: #e9ecef;
    border-color: #dee2e6;
}


</style>