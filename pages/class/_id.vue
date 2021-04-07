<template>
  <div class="container m-5">
    <div>
      <h1 class="title">Libro de Clases</h1>
      <b-button variant="primary" size="lg" class="mb-5 mt-4" v-b-modal.newquestion>NUEVA PREGUNTA</b-button>
      
      <b-modal id="newquestion" size="xl" title="Nueva Pregunta" ok-title="Guardar" cancel-title="Cancelar">
        <label>Ingresa el texto de la pregunta. Una vez guardada podrás agregar las respuestas, imágenes e indicadores de aprendizaje</label>
        <textarea class="w-100" rows="5"></textarea>
      </b-modal>
      
      <b-card title="Preguntas en ejecución">
        <div v-if="questions.executingQuestions.length === 0">
          No hay preguntas en ejecución
        </div>
        <div v-for="(e,i) in questions.executingQuestions" :key="i">
          <h5 class="mt-2">{{i+1}}) {{e.text}}</h5>
          <b-button-group class="w-100">
            <b-button variant="dark" v-on:click="close(e.id)">Cerrar Pregunta</b-button>
            <b-button variant="secondary" v-on:click="cancel(e.id)">Volver a estado Pendiente</b-button>
          </b-button-group>
        </div>
      </b-card>
      <b-card title="Preguntas pendientes">
        <div v-if="questions.pendingQuestions.length === 0">
          No hay preguntas pendientes
        </div>
        <div v-for="(e,i) in questions.pendingQuestions" :key="i">
          <h5 class="mt-2">{{i+1}}) {{e.text}}</h5>
          <b-button-group class="w-100">
            <b-button variant="success" v-on:click="execute(e.id)">Ejecutar</b-button>
            <b-button variant="primary" :href="`/editquestion/${e.id}`">Editar</b-button>
            <b-button variant="danger">Eliminar</b-button>
          </b-button-group>
        </div>
      </b-card>
      <b-card title="Preguntas cerradas">
        <div v-if="questions.oldQuestions.length === 0">
          No hay preguntas cerradas
        </div>
        <div v-for="(e,i) in questions.oldQuestions" :key="i">
          <h5 class="mt-2">{{i+1}}) {{e.text}}</h5>
          <b-button-group class="w-100">
            <b-button variant="light">Ver Estadísticas</b-button>
            <b-button variant="primary" v-on:click="execute(e.id)">Volver a Ejecución</b-button>
          </b-button-group>
        </div>
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

      questions: {
        executingQuestions: [],
        oldQuestions: [],
        pendingQuestions: []
      }

    }
  },

  created() {
    this.getQuestions()
  },

  methods: {

    execute(id) {
      fetch(this.endpoint + `executeQuestion/${id}`, {
        method: 'PUT', // or 'PUT'
        mode: 'cors',
        headers:{
          'Content-Type': 'application/json',
          'x-lum-token': localStorage.lumkey
        },

      })
      .then(res => res.json())
      .catch(error => console.error('Error:', error))
      .then(response => {
        if (response.internalCode === 200) {
          this.getQuestions()
        }

        if (response.internalCode === 401) {
          this.$router.push('/login')
        }
      });
    },

    close(id) {
      fetch(this.endpoint + `closeQuestion/${id}`, {
        method: 'PUT', // or 'PUT'
        mode: 'cors',
        headers:{
          'Content-Type': 'application/json',
          'x-lum-token': localStorage.lumkey
        },

      })
      .then(res => res.json())
      .catch(error => console.error('Error:', error))
      .then(response => {
        if (response.internalCode === 200) {
          this.getQuestions()
        }

        if (response.internalCode === 401) {
          this.$router.push('/login')
        }
      });
    },

    cancel(id) {
      fetch(this.endpoint + `cancelQuestion/${id}`, {
        method: 'PUT', // or 'PUT'
        mode: 'cors',
        headers:{
          'Content-Type': 'application/json',
          'x-lum-token': localStorage.lumkey
        },

      })
      .then(res => res.json())
      .catch(error => console.error('Error:', error))
      .then(response => {
        if (response.internalCode === 200) {
          this.getQuestions()
        }

        if (response.internalCode === 401) {
          this.$router.push('/login')
        }
      });
    },

    getQuestions() {
      let id = this.$route.params.id

      fetch(this.endpoint + `book/${id}/questions`, {
        method: 'GET', // or 'PUT'
        mode: 'cors',
        headers:{
          'Content-Type': 'application/json',
          'x-lum-token': localStorage.lumkey
        },

      })
      .then(res => res.json())
      .catch(error => console.error('Error:', error))
      .then(response => {
        if (response.internalCode === 200) {
          this.questions = response.payload
        }

        if (response.internalCode === 401) {
          this.$router.push('/login')
        }
      });
    }
  }
}
</script>

<style>

</style>
