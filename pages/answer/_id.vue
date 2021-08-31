<template>
  <div class="container m-5">
    <div>
      <b-spinner v-if="answering"></b-spinner>
    </div>
    <div v-if="pending">
      <!-- {{ question }} -->
      <!-- <Logo /> -->
      <!-- <h1 class="title">Bienvenido, Danny</h1> -->
      <hr>
      <div class="small">{{question.book.name}} / {{question.book.area}}</div>
      <h4>{{question.text}}</h4>
      <img :src="question.image" style="width: 320px; max-width: 100%">
      <hr>
      <div v-for="(e,i) in question.possible_answers" :key="i">
        <b-button block v-on:click="answer(i)" class="mb-4">
          <img :src="e.image" style="width: 120px; max-width: 100%">
          <span>{{e.text}}</span>
        </b-button>
      </div>
    </div>

    <b-modal id="modalAnswer"
      title="Confirma tu respuesta"
      cancel-title="Cancelar"
      ok-title="Confirmar"
      @ok = "sendAnswer()"
    >
      <p class="lead my-4">{{question.text}}</p>
      <hr>
      <label class="small">Mi respuesta:</label>
      <p class="h4">{{possible_answer.text}}</p>
      <p class=""><img :src="possible_answer.image" style="width:120px; max-width: 100%"></p>
    </b-modal>
  </div>
</template>

<script>
import config from '~/config.js'

export default {
  data() {
    return {
      pending: true,
      answering: false,
      endpoint: config.endpoint,

      question: {
        book: {}
      },

      possible_answer: {}
    }
  },

  created() {
    this.getQuestion(this.$route.params.id)
  },

  methods: {
    getQuestion(id) {
      let url = this.endpoint + `questions/${id}`;
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
          console.log(response)
          this.question = response.payload
        }
      });
    },
    answer(i) {
      this.possible_answer = this.question.possible_answers[i]
      this.$root.$emit('bv::show::modal', 'modalAnswer')
    },

    sendAnswer() {
      let data = {
        idQuestion: this.question.id,
        idAnswer: this.possible_answer.id,
        idStudent: 100
      }
      
      let url = this.endpoint + `answers`
      
      fetch(url, {
        method: "POST",
        mode: "cors",
        headers: {
          "Content-Type": "application/json",
          "x-lum-token": localStorage.lumkey,
        },
        body: JSON.stringify(data)
      })
      .then((res) => res.json())
      .catch((error) => console.error("Error:", error))
      .then((response) => {
        if (response.internalCode === 200) {
          console.log(response)
        }
      });
    }
  }
}
</script>

<style>

</style>
