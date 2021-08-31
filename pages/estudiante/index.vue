<template>
  <div class="container m-5">
    <div>
      <!-- <Logo /> -->
      <h1 class="title">Bienvenido, {{dashboard.user.name}}</h1>
      <!-- {{dashboard.classes}} -->
      <hr>
      <h2>Mis preguntas activas</h2>
      <hr>
      <div class="mb-4" v-for="(e,i) in dashboard.questions" :key="i">
        <b-button variant="primary" block :href="'/answer/' + e.id">{{e.text}}</b-button>
      </div>
    </div>
    {{dashboard.questions}}
  </div>
</template>

<script>
import config from '~/config.js'

export default {
  data() {
    return {
      endpoint: config.endpoint,

      dashboard: {
        user: {},
        questions: []
      }
    }
  },

  created() {
    // console.log('config', config)
    this.getDashboard()
  },

  methods: {
    getDashboard() {
      fetch(this.endpoint + 'studentdashboard', {
        method: 'GET', // or 'PUT'
        mode: 'cors',
        headers:{
          'Content-Type': 'application/json',
          'x-lum-token': localStorage.lumkey
        },

      }).then(res => res.json())
        .catch(error => console.error('Error:', error))
        .then(response => {
          console.log(response)
          if (response.internalCode === 200) {
            this.dashboard = response.payload.dashboard
          }

          if (response.internalCode === 401) {
            this.$router.push('login')
          }
        });
    }
  }
}
</script>

<style>

</style>
