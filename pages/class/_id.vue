<template>
  <div class="container m-5">
      <div><a href="/dashboard">Volver al Dashboard</a></div>
      <h1>{{theclass.name}} ID: ({{theclass.id}})</h1>
      <div>Nivel: {{theclass.level}} | Institución: {{theclass.idSchool}}</div>
      <!-- {{ theclass }} -->
      <hr>
      <h2>Estudiantes</h2>
      <div v-for="(e,i) in theclass.students" :key="i">
          {{e.name}} {{e.lastname}} <span v-if="e.rut">({{e.rut}})</span>
      </div>
      <div v-if="!theclass.students.length" class="mt-4">Aún no hay estudiantes agregados a esta clase</div>
        <!-- {{theclass}} -->
  </div>
</template>

<script>
import config from '~/config.js'

export default {
  data() {
    return {
        endpoint: config.endpoint,

        theclass: {
            students: []
        }
    }
  },

  created() {
      this.getClass()
  },

  methods: {
    getClass() {
        console.log(this.endpoint + 'class/' + this.$route.params.id)
      fetch(this.endpoint + 'class/' + this.$route.params.id, {
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
            this.theclass = response.payload
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
