<template>
    <!-- navbar component disini -->
    <NavBar :name="userName" :role="roleId" />
    <div>
        <div class="container mt-5">
            <div class="row">
                <div class="col-md-3 mb-3">
                    <div class="mb-3">
                        <label for="month" class="form-label">Month :</label>
                        <select class="form-control form-select" id="month" v-model="month" @change="getReport()">
                            <option value>Choose Month...</option>
                            <option v-for="month in months" :key="month.value" :value="month.value">{{ month.name }}
                            </option>
                        </select>
                    </div>
                </div>
                <div class="col-md-3 mb-3">
                    <div class="mb-3">
                        <label for="date" class="form-label">Date :</label>
                        <input type="date" class="form-control" id="date" v-model="date" @change="handleDateChange"
                            placeholder="masuian tanggal">
                    </div>
                </div>

                <div class="col-md-3 mb-3">
                    <div class="mb-3">
                        <label for="weeks" class="form-label">Week :</label>
                        <select class="form-control form-select" id="weeks" v-model="weeks" @change="getReport()">
                            <option value>Comming...</option>
                            <option v-for="week in weeks" :key="week.value" :value="week.value">{{ week.name }}</option>
                        </select>
                    </div>
                </div>
            </div>
        </div>

        <!-- The rest of your code remains unchanged -->


        <div class="container mt-5">
            <div class="row justify-content-center">
                <div class="col-12 col-sm-4 mb-3">
                    <div class="box border-left-blue">
                        <div>
                            <label>Order Count</label>
                            <h2>{{ data.orderCount }}</h2>
                        </div>
                        <div class="icon">
                            <i class="fa-solid fa-square-poll-vertical"></i>
                            <!-- Sesuaikan dengan ikon yang diinginkan -->
                        </div>
                    </div>
                </div>
                <div class="col-12 col-sm-4 mb-3">
                    <div class="box border-left-green">
                        <div>
                            <label>Min Payment</label>
                            <h2>Rp {{ data.minPayment }}</h2>
                        </div>
                        <div class="icon">
                            <i class="fa-solid fa-minus"></i> <!-- Sesuaikan dengan ikon yang diinginkan -->
                        </div>
                    </div>
                </div>
                <div class="col-12 col-sm-4 mb-3">
                    <div class="box border-left-yellow">
                        <div>
                            <label>Max Payment</label>
                            <h2>Rp {{ data.maxPayment }}</h2>
                        </div>
                        <div class="icon">
                            <i class="fa-solid fa-plus"></i> <!-- Sesuaikan dengan ikon yang diinginkan -->
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <table class="table table-striped table-hover">
            <thead>
                <tr>
                    <th>No</th>
                    <th>Customer Name</th>
                    <th>Table No</th>
                    <th>Order Date</th>
                    <th>Time</th>
                    <th>Status</th>
                    <th>Total</th>
                    <th>Waitress</th>
                    <th>Cashier</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="(order, index) in data.orders" :key="index">
                    <td>{{ index + 1 }}</td>
                    <td>{{ order.customer_name }}</td>
                    <td>{{ order.table_no }}</td>
                    <td>{{ order.order_date }}</td>
                    <td>{{ order.order_time }}</td>
                    <td>{{ order.status }}</td>
                    <td>{{ 'Rp ' + order.total }}</td>
                    <td>{{ order.waitress ? order.waitress.name : '-' }}</td>
                    <td>{{ order.cashier ? order.cashier.name : '-' }}</td>

                </tr>
            </tbody>
        </table>




    </div>
</template>

<script>
import NavBar from '@/components/NavBar.vue';
import router from '@/router';
import axios from 'axios';

export default {
    components: {
        NavBar
    },

    data() {
        return {
            userName: "",
            roleId: "",
            months: [
                { value: 1, name: 'January' },
                { value: 2, name: 'February' },
                { value: 3, name: 'March' },
                { value: 4, name: 'April' },
                { value: 5, name: 'May' },
                { value: 6, name: 'June' },
                { value: 7, name: 'July' },
                { value: 8, name: 'August' },
                { value: 9, name: 'September' },
                { value: 10, name: 'October' },
                { value: 11, name: 'November' },
                { value: 12, name: 'December' },
            ],
            month: "",
            date: "",
            data: {
                orderCount: 0,
                minPayment: 0,
                maxPayment: 0
            }
        }
    },

    mounted() {
        this.userName = localStorage.getItem('name');
        this.roleId = localStorage.getItem('role_id');

        if (!this.userName || this.userName === '' || this.userName === null) {
            router.push({ name: 'login' });
        }


    },


    methods: {
        getReport() {
            axios.get('http://localhost/restaurant_order_api/public/api/order-report-month?month=' + this.month, {
                headers: {
                    "Authorization": `Bearer ${localStorage.getItem('token')}`
                }
            })
                .then((response) => {
                    console.log(response.data.data); // Debug: memastikan data diterima
                    this.data = response.data.data;
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

        getReportDate() {
            axios.get('http://localhost/restaurant_order_api/public/api/order-report-date?date=' + this.date, {
                headers: {
                    "Authorization": `Bearer ${localStorage.getItem('token')}`
                }
            })
                .then((response) => {
                    console.log(response.data.data); // Debug: memastikan data diterima
                    this.data = response.data.data;
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

        handleDateChange() {
            this.getReportDate();
            this.date = ""; // Mengosongkan nilai tanggal setelah laporan diambil
        },



    }
}
</script>

<style>
.container .row {
    display: flex;
    justify-content: center;
    /* Rata tengah secara horizontal */
    align-items: center;
    /* Rata tengah secara vertikal */

}

.box {
    display: flex;
    justify-content: space-between;
    align-items: center;
    border-radius: 8px;
    padding: 20px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    background-color: white;
    position: relative;
    min-width: 250px;
    /* Ukuran minimum agar terlihat seragam */
}

.box label {
    display: block;
    font-size: 14px;
    font-weight: 600;
    color: #5a5c69;
}

.box h2 {
    margin: 0;
    font-size: 24px;
    font-weight: bold;
    color: #333;
}

.border-left-blue {
    border-left: 4px solid #4e73df;
}

.border-left-green {
    border-left: 4px solid #1cc88a;
}

.border-left-yellow {
    border-left: 4px solid #f6c23e;
}

.icon {
    font-size: 2rem;
    color: #d1d3e2;
}
</style>