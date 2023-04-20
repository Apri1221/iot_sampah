<!-- Please remove this file from your project -->
<template>
    <div class="overflow-hidden bg-white py-24 sm:py-32">
        <div class="mx-auto max-w-7xl px-6 lg:px-8">
            <div
                class="mx-auto grid max-w-2xl grid-cols-1 gap-y-16 gap-x-8 sm:gap-y-20 lg:mx-0 lg:max-w-none lg:grid-cols-2">
                <div class="lg:pr-8 lg:pt-4">
                    <div class="lg:max-w-lg">
                        <h2 class="text-lg font-semibold leading-8 tracking-tight text-indigo-600">Detecting</h2>
                        <p class="mt-2 text-3xl font-bold tracking-tight text-gray-900 sm:text-4xl">Masukkan sampah Anda
                        </p>
                        <p class="mt-6 text-lg leading-8 text-gray-600">Sistem akan membaca sampah yang Anda masukkan.
                            Tetap berada pada halaman ini bila masih akan memasukkan sampah, atau tekan tombol 'Selesai'
                            apabila sudah tidak ada sampah yang akan dimasukkan</p>
                    </div>
                </div>
                <div class="relative flex flex-col justify-center min-w-screen sm:items-center sm:pt-0">
                    <!-- <div class="shadow-xl ring-1 ring-gray-400/10" style="width: 30rem;"> -->
                    <p class="font-bold text-lg" style="margin-top: 20px;">{{ dataUser.name }}</p>
                    <p class="text-sm mt-2">Saldo: {{ formatRupiah(dataUser.saldo, true) }}</p>
                    <br>
                    <div class="flex content-end items-end">
                        <p class="mt-2 text-4xl font-bold tracking-tight sm:text-5xl" :class="{ 'text-gray-900': received_messages >= 0, 'text-red-700': received_messages < 0 }">{{ received_messages > 0 ? `+${received_messages}` : received_messages }}
                        </p>
                        <p class="">({{ dataUser.point }})</p>
                    </div>

                    <button type="submit" @click="redirectDashboard()"
                        class="mt-12 group relative flex justify-center rounded-md border border-transparent bg-indigo-600 py-3 px-8 text-sm font-medium text-white hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2">
                        Selesai
                    </button>
                    <!-- </div> -->
                </div>
                <!-- <img src="https://tailwindui.com/img/component-images/dark-project-app-screenshot.png" alt="Product screenshot" class="w-[48rem] max-w-none rounded-xl shadow-xl ring-1 ring-gray-400/10 sm:w-[57rem] md:-ml-4 lg:-ml-0" width="2432" height="1442" /> -->
            </div>
            <transition name="fade">
                <div v-if="isModalVisible">
                    <div @click="onToggle" class="absolute bg-black opacity-70 inset-0 z-0"></div>
                    <div class="w-full max-w-lg p-3 relative mx-auto my-auto rounded-xl shadow-lg bg-white">
                        <div>
                            <div class="text-center p-3 flex-auto justify-center leading-6">
                                <svg xmlns="http://www.w3.org/2000/svg"
                                    class="w-16 h-16 flex items-center text-red-500 mx-auto" viewBox="0 0 20 20"
                                    fill="currentColor">
                                    <path fill-rule="evenodd"
                                        d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z"
                                        clip-rule="evenodd" />
                                </svg>
                                <h2 class="text-2xl font-bold py-4">Ooopss!</h2>
                                <p class="text-md text-gray-600 px-8">
                                    Kamu terdeteksi melakukan tindak kecurangan!
                                </p>
                            </div>
                            <div class="p-3 mt-2 text-center space-x-4 md:block">
                                <!-- <button
                                    class="mb-2 md:mb-0 bg-white px-5 py-2 text-sm shadow-sm font-medium tracking-wider border text-gray-600 rounded-md hover:shadow-lg hover:bg-gray-100">
                                    Save
                                </button> -->
                                <button @click="onToggle"
                                    class="mb-2 md:mb-0 bg-red-500 border border-red-500 px-5 py-2 text-sm shadow-sm font-medium tracking-wider text-white rounded-md hover:shadow-lg hover:bg-red-600">
                                    Mengerti
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            </transition>
        </div>
    </div>
</template>

<!-- <script setup>
import { LockClosedIcon, TrashIcon } from '@heroicons/vue/20/solid'

</script> -->

<script>
import SockJS from "sockjs-client";
import Stomp from "webstomp-client";


export default {
    name: 'Profile',
    props: ["clientId", "dataUser"],
    data() {
        return {
            received_messages: 0,
            send_message: null,
            connected: false,
            isOpen: false
        };
    },

    computed: {
        isModalVisible() {
            return this.isOpen;
        }
    },

    methods: {
        redirectDashboard() {
            let dataTempUser = {...this.dataUser}
            dataTempUser.point = this.received_messages;
            this.$axios.$post('/api/users/update', dataTempUser).then(() => {
                this.$router.push('/');
            })
        },
        send() {
            console.log("Send message:" + this.send_message);
            if (this.stompClient && this.stompClient.connected) {
                const msg = { name: this.send_message };
                this.stompClient.send("/app/hello", JSON.stringify(msg), {});
            }
        },
        connect() {
            this.socket = new SockJS("https://sheltered-wave-88873.herokuapp.com/broker");
            this.stompClient = Stomp.over(this.socket);
            this.stompClient.connect({}, frame => {
                this.connected = true;
                console.log('Connected: ' + frame + "\nClientId: " + this.clientId);
                this.stompClient.subscribe("/topic/pushmessages/" + this.clientId, tick => {
                    console.log(tick, JSON.parse(tick.body));

                    if (this.isOpen) return;
                    // this.received_messages.push(JSON.parse(tick.body).text);
                    this.received_messages += Number(JSON.parse(tick.body).text);
                    if (Number(JSON.parse(tick.body).text) < 0) this.isOpen = true;
                });
            },
                error => {
                    console.log(error);
                    this.connected = false;
                }
            );
        },
        formatRupiah(angka, prefix){
            const str_angka = angka.toString();
            let separator;
            var number_string = str_angka.replace(/[^,\d]/g, '').toString(),
            split   		= number_string.split(','),
            sisa     		= split[0].length % 3,
            rupiah     		= split[0].substr(0, sisa),
            ribuan     		= split[0].substr(sisa).match(/\d{3}/gi);
        
            // tambahkan titik jika yang di input sudah menjadi angka ribuan
            if(ribuan){
                separator = sisa ? '.' : '';
                rupiah += separator + ribuan.join('.');
            }
        
            rupiah = split[1] != undefined ? rupiah + ',' + split[1] : rupiah;
            return prefix == undefined ? rupiah : (rupiah ? 'Rp. ' + rupiah : '');
        },
        disconnect() {
            if (this.stompClient) {
                this.stompClient.disconnect();
            }
            this.connected = false;
        },
        tickleConnection() {
            this.connected ? this.disconnect() : this.connect();
        },
        onToggle() {
            this.isOpen = !this.isOpen;
            this.redirectDashboard();
        }
    },
    mounted() {
        // if (this.clientId != null) {
            this.disconnect()
            console.log(this.dataUser)
            this.connect()
        // }
    }
}
</script>

<style>
.reader__scan_region video {
    width: 30rem !important;
}

#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
}

.fade-enter,
.fade-leave-to {
    opacity: 0;
}

.fade-enter-active,
.fade-leave-active {
    transition: opacity 500ms ease-out;
}
</style>
