<template>
  <div v-if="clientId != null">
    <Profile :clientId="clientId" :dataUser="dataUser"/>
  </div>
</template>

<script>
import Profile from '~/components/Profile.vue';


export default {
    name: "IndexPage",
    components: { Profile },
    data() {
      return {
        clientId: null,
        dataUser: null,
      }
    },
    async mounted() {
      // this.clientId = await this.$axios.$get('/api/subscribe?topic=/node-jarak-r,/node-ir-r&school_id=1')
      this.dataUser = await this.$axios.$get('/api/users/' + this.$route.query.user)
      this.clientId = await this.$axios.$get('/api/subscribe?school_id=' + this.dataUser.school.id)
    }
}
</script>
