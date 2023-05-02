<template>
  <!-- <div v-if="clientId != null"> -->
    <Scan :clientId="clientId"/>
  <!-- </div> -->
</template>

<script>
import Scan from '~/components/Scan.vue';

export default {
    name: "Index",
    components: { Scan },
    data() {
      return {
        clientId: null,
      }
    },
    async mounted() {
      // this.clientId = await this.$axios.$get('/api/subscribe?topic=/node-jarak-r') // Hardcode
    }
}
</script>
