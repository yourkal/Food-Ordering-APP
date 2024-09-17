<template lang="">
     <!-- navbar component disini -->
     <NavBar :name="userName" :role="roleId" />
    <div>
        <div class="container-order-detail mt-5">
            <div class="table-responsive">
            <table class="table table-bordered table-striped table-hover">
                <tbody>
                <tr>
                    <td>Customer Name : {{ order.customer_name }}</td>
                    <td>Table No : {{ order.table_no }}</td>
                    <td>Order Date : {{ order.order_date }}</td>
                    <td>Status : {{ order.status }}</td>
                </tr>
                <tr>
                    <td>Waitress : {{ order.waitress ? order.waitress.name: 'tidak ada waitress' }}</td>
                    <td>Cashier : {{order.cashier ? order.cashier.name: 'tidak ada cashier'}}</td>
                    <td>Order Time : {{ order.order_time}}</td>
                    <td>Grand Total : {{ order.total}}</td>
                </tr>
                </tbody>
            </table>
            </div>

            <hr class="my-5">

            <table class="table table-bordered table-striped table-hover">
                <thead>
                    <tr>
                        <th>No</th>
                        <th>Product Name</th>
                        <th>Price</th>
                        <th>Quantity</th>
                        <th>Total</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(item, index) in order.order_detail" :key="index">
                        <td>{{ index + 1 }}</td>
                        <td>{{ item.item ? item.item.name : "tidak ada item" }}</td>
                        <td>Rp {{ item.price }}</td>
                        <td>{{ item.qty }}</td>
                        <td>Rp {{ item.price * item.qty }}</td>
                    </tr>
                </tbody>              
            </table>

            <!-- button done if user is chef and order.status == ordered -->
            <!-- button paid if user is cashier or manager and order.status == done  -->

            <div class="mt-3">
                <button v-if="(order.status == 'ordered') && (roleId == 2)" class="btn btn-primary" @click="setAsDone(order.id)"> Done </button>
                <button v-if="(order.status == 'done') && (roleId == 3 || roleId == 4)" class="btn btn-success" @click="setAsPaid(order.id)"> Paid </button>
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
            order: ""
        }
    },

    mounted() {
        this.userName = localStorage.getItem('name');
        this.roleId = localStorage.getItem('role_id');

        if (!this.userName || this.userName === '' || this.userName === null) {
            router.push({ name: 'login' });
        }

        this.getOrder()
    },

    methods: {
        getOrder() {
            axios.get('http://localhost/restaurant_order_api/public/api/order/' + this.$route.params.orderId, {
                headers: {
                    "Authorization": `Bearer ${localStorage.getItem('token')}`
                }
            })
                .then((response) => {
                    console.log(response.data.data); // Debug: memastikan data diterima
                    this.order = response.data.data;
                })
                .catch((error) => {
                    if (error.response.status == 401) {
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

        setAsDone(orderId) {
            // Menampilkan konfirmasi pertama sebelum mengirim permintaan ke server
            const swalWithBootstrapButtons = Swal.mixin({
                customClass: {
                    confirmButton: "btn btn-success",
                    cancelButton: "btn btn-danger"
                },
                buttonsStyling: false
            });

            swalWithBootstrapButtons.fire({
                title: "Are you sure you want to set this order as done?",
                text: "This action cannot be reverted!",
                icon: "warning",
                showCancelButton: true,
                confirmButtonText: "Yes, set to done!",
                cancelButtonText: "No, cancel!",
                reverseButtons: true
            }).then((result) => {
                if (result.isConfirmed) {
                    // Jika pengguna mengonfirmasi, kirim permintaan ke server untuk mengubah status menjadi "paid"
                    axios.get('http://localhost/restaurant_order_api/public/api/order/' + orderId + '/set-as-done', {
                        headers: {
                            "Authorization": `Bearer ${localStorage.getItem('token')}`
                        }
                    })
                        .then((response) => {
                            if (response.status == 200) {
                                swalWithBootstrapButtons.fire({
                                    title: "Done!",
                                    text: "The order has been set to done.",
                                    icon: "success"
                                });
                                this.order = response.data.data; // Update order data
                            }
                        })
                        .catch((error) => {
                            if (error.response.status == 401) {
                                localStorage.removeItem('token');
                                localStorage.removeItem('email');
                                localStorage.removeItem('name');
                                localStorage.removeItem('role_id');
                                router.push({ name: 'login' });
                            } else if (error.response.status == 403) {
                                swalWithBootstrapButtons.fire({
                                    title: "Error",
                                    text: "Order cannot be set to paid because the status is not ordered!",
                                    icon: "error"
                                });
                            } else {
                                console.log(error);
                                console.log('Error updating order status');
                            }
                        });
                } else if (result.dismiss === Swal.DismissReason.cancel) {
                    // Jika pengguna membatalkan, tidak melakukan apa-apa
                    swalWithBootstrapButtons.fire({
                        title: "Cancelled",
                        text: "The order status remains unchanged.",
                        icon: "info"
                    });
                }
            });
        },

        setAsPaid(orderId) {
            // Menampilkan konfirmasi pertama sebelum mengirim permintaan ke server
            const swalWithBootstrapButtons = Swal.mixin({
                customClass: {
                    confirmButton: "btn btn-success",
                    cancelButton: "btn btn-danger"
                },
                buttonsStyling: false
            });

            swalWithBootstrapButtons.fire({
                title: "Are you sure you want to set this order as paid?",
                text: "This action cannot be reverted!",
                icon: "warning",
                showCancelButton: true,
                confirmButtonText: "Yes, set to paid!",
                cancelButtonText: "No, cancel!",
                reverseButtons: true
            }).then((result) => {
                if (result.isConfirmed) {
                    // Jika pengguna mengonfirmasi, kirim permintaan ke server untuk mengubah status menjadi "paid"
                    axios.get('http://localhost/restaurant_order_api/public/api/order/' + orderId + '/set-as-paid', {
                        headers: {
                            "Authorization": `Bearer ${localStorage.getItem('token')}`
                        }
                    })
                        .then((response) => {
                            if (response.status == 200) {
                                swalWithBootstrapButtons.fire({
                                    title: "Paid!",
                                    text: "The order has been set to paid.",
                                    icon: "success"
                                });
                                this.order = response.data.data; // Update order data
                            }
                        })
                        .catch((error) => {
                            if (error.response.status == 401) {
                                localStorage.removeItem('token');
                                localStorage.removeItem('email');
                                localStorage.removeItem('name');
                                localStorage.removeItem('role_id');
                                router.push({ name: 'login' });
                            } else if (error.response.status == 403) {
                                swalWithBootstrapButtons.fire({
                                    title: "Error",
                                    text: "Order cannot be set to paid because the status is not done!",
                                    icon: "error"
                                });
                            } else {
                                console.log(error);
                                console.log('Error updating order status');
                            }
                        });
                } else if (result.dismiss === Swal.DismissReason.cancel) {
                    // Jika pengguna membatalkan, tidak melakukan apa-apa
                    swalWithBootstrapButtons.fire({
                        title: "Cancelled",
                        text: "The order status remains unchanged.",
                        icon: "info"
                    });
                }
            });
        }
    }
}
</script>

<style>
/* .table th, .table td {
    vertical-align: middle;
} */
</style>