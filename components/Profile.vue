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
                    <p class="">{{ dataUser.name }}</p>
                    <br>
                    <div class="flex content-end items-end">
                        <p class="mt-2 text-4xl font-bold tracking-tight text-gray-900 sm:text-5xl">+{{ received_messages }}
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
            connected: false
        };
    },
    methods: {
        redirectDashboard() {
            let dataTempUser = {...this.dataUser}
            dataTempUser.point += this.received_messages;
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
            this.socket = new SockJS("http://localhost:8080/broker");
            this.stompClient = Stomp.over(this.socket);
            this.stompClient.connect({}, frame => {
                this.connected = true;
                console.log('Connected: ' + frame + "\nClientId: " + this.clientId);
                this.stompClient.subscribe("/topic/pushmessages/" + this.clientId, tick => {
                    console.log(tick, JSON.parse(tick.body));
                    // this.received_messages.push(JSON.parse(tick.body).text);
                    this.received_messages += 1
                });
            },
                error => {
                    console.log(error);
                    this.connected = false;
                }
            );
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
</style>
