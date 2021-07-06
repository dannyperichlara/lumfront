<template>
  <div class="container m-5">
    <div>
      <!-- <Logo /> -->
      <h1 class="title">Bienvenido, {{dashboard.user.name}}</h1>
      <!-- {{dashboard.classes}} -->
      <hr>
      <h2>Mis cursos</h2>
      <b-card class="mt-4" v-for="(e,i) in dashboard.classes":key="i">
        <h2 class="mt-4">{{e.subject}}</h2>
        <h5>{{e.class}}</h5>
        <a :href="`class/${e.idClass}`">Editar Curso / Agregar Alumnos</a>
        <b-button-group class="w-100 mt-4">
          <b-button variant="primary" :href="`book/${e.id}`">Ver Preguntas</b-button>
        </b-button-group>
      </b-card>
    </div>
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
        classes: []
      }
    }
  },

  created() {
    // console.log('config', config)
    this.getDashboard()
  },

  methods: {
    getDashboard() {
      fetch(this.endpoint + 'dashboard', {
        method: 'GET', // or 'PUT'
        mode: 'cors',
        headers:{
          'Content-Type': 'application/json',
          'x-lum-token': localStorage.lumkey
        },

      }).then(res => res.json())
        .catch(error => console.error('Error:', error))
        .then(response => {
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
