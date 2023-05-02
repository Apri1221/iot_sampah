<!-- Please remove this file from your project -->
<template>
  <div class="overflow-hidden bg-white py-24 sm:py-32">
    <div class="mx-auto max-w-7xl px-6 lg:px-8">
      <div class="mx-auto grid max-w-2xl grid-cols-1 gap-y-16 gap-x-8 sm:gap-y-20 lg:mx-0 lg:max-w-none lg:grid-cols-2">
        <div class="lg:pr-8 lg:pt-4">
          <div class="lg:max-w-lg">
            <h2 class="text-lg font-semibold leading-8 tracking-tight text-indigo-600">Scanning</h2>
            <p class="mt-2 text-3xl font-bold tracking-tight text-gray-900 sm:text-4xl">Scan QRCode Anda</p>
            <p class="mt-6 text-lg leading-8 text-gray-600">Posisikan QRCode pada kamera sehingga sistem akan mengenali
              Anda</p>
          </div>
          <div class="water"></div>
        </div>
        <div class="relative flex items-top justify-center min-w-screen sm:items-center sm:pt-0">
          <div id="reader" width="600" class="shadow-xl ring-1 ring-gray-400/10" style="width: 30rem;"></div>
        </div>
        <!-- <img src="https://tailwindui.com/img/component-images/dark-project-app-screenshot.png" alt="Product screenshot" class="w-[48rem] max-w-none rounded-xl shadow-xl ring-1 ring-gray-400/10 sm:w-[57rem] md:-ml-4 lg:-ml-0" width="2432" height="1442" /> -->
      </div>
    </div>
  </div>
</template>

<script>
// To use Html5QrcodeScanner (more info below)
import { Html5QrcodeScanner } from "html5-qrcode"

// To use Html5Qrcode (more info below)
import { Html5Qrcode } from "html5-qrcode"

import SockJS from "sockjs-client";
import Stomp from "webstomp-client";

import Paho from 'paho-mqtt'

var client;


export default {
  name: 'Scan',
  // props: ["clientId"],
  data() {
    return {
      html5QrcodeScanner: null,
      html5QrCode: null,
      color: "-120%",
      received_messages: "-100%",
      bg_color: "#8ddbf5",
      mqtt_server: 'tailor.cloudmqtt.com',
      mqtt_port: 36503,
      publish_topic: '/breez_test/message',
      subscribe_topic: '/node-jarak-r',
      publish_message: 'Web client: Hello World!'
    };
  },
  methods: {
    onScanSuccess(decodedText, decodedResult) {
      // handle the scanned code as you like, for example:
      // ...
      // this.html5QrcodeScanner.clear();
      console.log(`Code matched = ${decodedText}`, decodedResult);
      this.html5QrCode.stop().then((ignore) => {
        this.$router.push({ name: 'profile', query: { user: decodedText } })
      }).catch((err) => {
        // Stop failed, handle it.
      });
    },
    onScanFailure(error) {
      // handle scan failure, usually better to ignore and keep scanning.
      // for example:
      console.warn(`Code scan error = ${error}`);
    },
    connect: function initClient() {
      let self = this;
      // Create a client instance
      let client_id =
        'breez-web-demo-' + Math.floor(Math.random() * 8999 + 1000);
      client = new Paho.Client(
        self.mqtt_server,
        self.mqtt_port,
        client_id
      );

      let willMessage = new Paho.Message(
        'Web Client - Disconnected ungracefully'
      );
      willMessage.destinationName = self.publish_topic;

      client.connect({
        onSuccess: () => {
          client.subscribe(self.subscribe_topic);

          let message = new Paho.Message('Web client connected!');
          message.destinationName = self.publish_topic;
          client.send(message);
        },
        useSSL: true,
        userName: "lvntsnrq",
        password: "79W8iNWE4G9i",
        willMessage: willMessage,
      });

      // set callback handlers
      client.onConnectionLost = onConnectionLost;
      client.onMessageArrived = onMessageArrived;

      // called when the client loses its connection
      function onConnectionLost(responseObject) {
        if (responseObject.errorCode !== 0) {
          console.log('onConnectionLost:' + responseObject.errorMessage);
        }
      }

      // called when a message arrives
      function onMessageArrived(message) {
        console.log('onMessageArrived:' + message.payloadString);
        self.received_messages = "-" + (32 - Number(message.payloadString)) + "%";
          // kalau 100% kosong
          if (Number(message.payloadString) > 60) self.bg_color = "#8ddbf5"
          else self.bg_color = "#f58d8d"
      }
    },
    disconnect: function () {
      client.disconnect();
    },
  },
  mounted() {
    this.connect()
    // this.html5QrcodeScanner = new Html5QrcodeScanner("reader", { fps: 20, qrbox: 250 }, /* verbose= */ false);
    // this.html5QrcodeScanner.render(this.onScanSuccess, this.onScanFailure);

    // This method will trigger user permissions
    Html5Qrcode.getCameras().then(devices => {
      if (devices && devices.length) {
        var cameraId = devices[0].id;
        // .. use this to start scanning.
        this.html5QrCode = new Html5Qrcode(/* element id */ "reader", false);
        const config = { fps: 15, qrbox: { width: 300, height: 300 } };
        this.html5QrCode.start({ deviceId: { exact: cameraId } }, config, this.onScanSuccess);
      }
    }).catch(err => {
      // handle err
    });
  }
}
</script>

<style>
.reader__scan_region video {
  width: 30rem !important;
}

/* https://codepen.io/Alhefel/pen/wvEXExL */
.water {
  width: 200px;
  height: 200px;
  margin: 50px auto;
  position: relative;
  background-color: v-bind(bg_color);
  box-shadow: inset 0 0 50px #1c637a;
  clip-path: polygon(0 0, 100% 0, 85% 100%, 15% 100%);
}

.water::before {
  content: "";
  position: absolute;
  width: 200%;
  height: 200%;
  top: v-bind(received_messages);
  left: -50%;
  background-color: #ececec;
  border-radius: 40%;
  animation: moving 12s linear infinite;
}

.water::after {
  content: "";
  position: absolute;
  width: 200%;
  height: 204%;
  top: v-bind(received_messages);
  left: -52%;
  background-color: #ececec80;
  border-radius: 40%;
  animation: moving 12s linear infinite;
  animation-delay: 0.5s;
}

@keyframes moving {
  0% {
    transform: rotate(0deg);
  }

  100% {
    transform: rotate(360deg);
  }
}
</style>
