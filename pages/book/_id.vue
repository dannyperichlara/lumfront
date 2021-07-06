<template>
  <div class="container m-5">
    <div>
      <div class="text-right"><a href="/dashboard">Volver al Dashboard</a></div>
      <h1 class="title">Libro de Clases</h1>
      <b-button variant="primary" size="lg" class="mb-5 mt-4" v-b-modal.newquestion>NUEVA PREGUNTA</b-button>
      
      <b-modal @ok="saveNewQuestion" id="newquestion" size="xl" title="Nueva Pregunta" ok-title="Siguiente" cancel-title="Cancelar" cancel-variant="danger">
        <label>PASO 1: Selecciona al menos un objetivo de aprendizaje</label>
        <b-list-group>
          <b-list-group-item
            v-for="(e, i) in oas"
            :key="i"
            v-if="e"
            :class="newQuestion.indicators.indexOf(e.id) != -1 ? 'p-2 small active mb-1' : 'p-2 small mb-1'
            "
            v-on:click="selectIndicator(e.id)"
          >
            {{ e.name }} Unidades: {{ e.unit }}
          </b-list-group-item>
        </b-list-group>
        <label class="mt-4">PASO 2: Escribe el texto de la pregunta</label>
        <textarea class="w-100" rows="5" v-model="newQuestion.text"></textarea>
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
            <b-button variant="danger" v-on:click="deleteQuestion(e.id)">Eliminar</b-button>
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
      },

      newQuestion: {
        text: '',
        idBook: this.$route.params.id,
        indicators: []
      },

      oas: []
    }
  },

  created() {
    this.getQuestions()
  },

  methods: {
    loadOAS(area, level) {
      let $this = this
      let url = this.endpoint + `oas/${area}/${level}`;
      fetch(url, {
        method: "GET", // or 'PUT'
        mode: "cors",
        headers: {
          "Content-Type": "application/json",
          "x-lum-token": localStorage.lumkey,
        },
      })
        .then((res) => res.json())
        .catch((error) => console.error("Error:", error))
        .then((response) => {
          if (response.internalCode === 200) {
            let oas = [null, { id: 1, name: "xxx1" }, { id: 2, name: "xxx2" }];

            for (let e of response.payload.hits) {
              oas[e.id] = e;
            }

            $this.oas = oas;
          }

          if (response.internalCode === 401) {
            this.$router.push("/login");
          }
        });
    },

    selectIndicator(id) {
      if (this.newQuestion.indicators.indexOf(id) == -1) {
        this.newQuestion.indicators.push(id);
      } else {
        this.removeIndicatorFromQuestion(id);
      }
    },

    removeIndicatorFromQuestion(id) {
      let filtered = this.newQuestion.indicators.filter((e) => e != id);
      this.newQuestion.indicators = filtered;
    },

    saveNewQuestion() {      
      fetch(this.endpoint + `newQuestion`, {
        method: 'POST', // or 'PUT'
        mode: 'cors',
        headers:{
          'Content-Type': 'application/json',
          'x-lum-token': localStorage.lumkey
        },
        body: JSON.stringify(this.newQuestion)
      })
      .then(res => res.json())
      .catch(error => console.error('Error:', error))
      .then(response => {
        if (response.internalCode === 200) {
          let newId = response.payload.questionId

          this.$router.push('/editquestion/' + newId)
        }

        if (response.internalCode === 401) {
          this.$router.push('/login')
        }
      });
    },

    deleteQuestion(id) {
      if (!confirm('¿Deseas borrar esta pregunta?')) return
      
      fetch(this.endpoint + `deleteQuestion/${id}`, {
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
            console.log(this.questions)
            this.loadOAS(
              this.questions.book.area,
              this.questions.book.level
            )
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
