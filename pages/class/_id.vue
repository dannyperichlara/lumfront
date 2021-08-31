<template>
  <div class="container m-5">
    <div><a href="/dashboard">Volver al Dashboard</a></div>
    <h1>{{theclass.name}} ID: ({{theclass.id}})</h1>
    <div>Nivel: {{theclass.level}} | Institución: {{theclass.idSchool}}</div>
    <hr>
    <b-button variant="primary" v-b-modal.addstudent> + Agregar estudiante</b-button>
    <hr>
    <h2>Estudiantes</h2>
    <div v-for="(e,i) in theclass.students" :key="i">
        {{e.name}} {{e.lastname}} <span v-if="e.rut">({{e.rut}})</span>
    </div>
    <div v-if="!theclass.students.length" class="mt-4">Aún no hay estudiantes agregados a esta clase</div>
    
    <b-modal id="addstudent" title="Agregar alumno" ok-title="Agregar alumno" cancel-title="Cancelar" @ok="addStudent()">
      <b-input class="mb-2" v-model="newStudent.rut" placeholder="RUT del alumno*"></b-input>
      <div v-if="newStudent.isnew" class="small mb-2">El alumno no se encuentra actualmente en la base de datos. Ingresa sus datos para agregarlo</div>
      <b-button v-on:click="findStudent()" v-if="!newStudent.isnew">Buscar Alumno</b-button>
      <b-input class="mb-2" v-model="newStudent.name" placeholder="Nombre del alumno*" v-if="newStudent.isnew"></b-input>
      <b-input class="mb-2" v-model="newStudent.lastname" placeholder="Apellido del alumno*" v-if="newStudent.isnew"></b-input>
      <!-- {{newStudent}} -->
    </b-modal>
  </div>
</template>

<script>
import config from '~/config.js'

export default {
  data() {
    return {
        endpoint: config.endpoint,

        newStudent: {
          rut: null,
          name: null,
          lastname: null,
          isnew: false
        },

        theclass: {
            students: []
        }
    }
  },

  created() {
      this.getClass()
  },

  methods: {
    findStudent() {
      // To do. Busca el alumno en la api y trae sus datos

      this.newStudent.isnew = true
    },

    addStudent() {
      if (this.newStudent.isnew) {
        let cleanRut = this.newStudent.rut.trim().split('-')[0].toString()

        let data = {
          rut: cleanRut,
          name: this.newStudent.name,
          lastname: this.newStudent.lastname,
          idClass: this.theclass.id,
          password: cleanRut.substr(0,4)
        }

        fetch(this.endpoint + 'students', {
          method: 'POST', // or 'PUT'
          mode: 'cors',
          headers:{
            'Content-Type': 'application/json',
            'x-lum-token': localStorage.lumkey
          },
          body: JSON.stringify(data)
        }).then(res => res.json())
          .catch(error => console.error('Error:', error))
          .then(response => {
            if (response.internalCode === 200) {
              this.newStudent = {
                rut: null,
                name: null,
                lastname: null,
                idClass: null,
                password: null
              }

              this.getClass()
            }

            if (response.internalCode === 401) {
              this.$router.push('login')
            }
        });
      }
    },

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
