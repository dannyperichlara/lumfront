<template>
  <div class="container m-5">
    <div>
      <h1 class="title">Editar Pregunta</h1>
      <p class="small">{{ question }}</p>
      <p class="small">{{ oas }}</p>
      <b-row class="mt-5" v-if="question">
        <b-col class="col-4">
          <div><label>Imagen de la pregunta (click para cambiar):</label></div>
          <img width="100%" :src="question.image" />
        </b-col>
        <b-col>
          <div><label>Texto de la pregunta:</label></div>
          <textarea
            rows="8"
            class="w-100 p-4 h5"
            v-model="question.text"
          ></textarea>
        </b-col>
      </b-row>

      <div class="mt-5 h2">
        <label>Indicadores asociados a la pregunta</label>
      </div>
      <b-list-group v-if="question && question.indicators && oas">
        <b-list-group-item
          class="h5"
          v-for="(indicator, j) in JSON.parse(question.indicators)"
          :key="j"
        >
          <b-row>
            <b-col v-if="oas[indicator]"
              >{{ j + 1 }}. {{ oas[indicator].name }}</b-col
            >
            <b-col class="text-right">
              <b-button variant="danger">x</b-button>
            </b-col>
          </b-row>
        </b-list-group-item>

        <b-button v-b-modal.indicators
          >Asociar nuevo indicador a la pregunta</b-button
        >
      </b-list-group>

      <b-modal id="indicators" size="xl" title="Indicadores">
        <p class="my-4">{{ oas }}</p>
      </b-modal>

      <div class="mt-5 h2"><label>Posibles Respuestas:</label></div>
      <b-list-group v-if="question && question.possible_answers">
        <b-list-group-item
          class="h5"
          v-for="(answer, j) in question.possible_answers"
          :key="j"
        >
          <b-row>
            <b-col class="col-1 text-nowrap"
              ><img height="60" :src="answer.image"
            /></b-col>
            <b-col>{{ answer.text }}</b-col>
            <b-col>{{ answer.indicators }}</b-col>
            <b-col class="text-nowrap">
              <span>Indicadores asociados:</span>
              <b-button
                class="ml-2"
                v-for="(indicator, j) in JSON.parse(question.indicators)"
                :key="j"
                :variant="
                  JSON.parse(answer.indicators).indexOf(1) != -1 ? 'primary' : 'secondary'
                "
              >
                <span>{{ j + 1 }}</span>
              </b-button>
            </b-col>
            <b-col class="text-right">
              <b-button variant="dark">Editar</b-button>
              <b-button variant="danger">x</b-button>
            </b-col>
          </b-row>
        </b-list-group-item>
        <b-button>Agregar nueva posible respuesta</b-button>
      </b-list-group>
    </div>
  </div>
</template>

<script>
import config from "~/config.js";

export default {
  data() {
    return {
      endpoint: config.endpoint,

      oas: new Array(800),

      question: {
        text: "",
        indicators: "[]",

        possibleAnswers: [],
      },
    };
  },

  created() {
    this.loadQuestion();
  },

  methods: {
    loadOAS(area, level) {
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
            let oas = [null, { name: "xxx1" }, { name: "xxx2" }];

            for (let e of response.payload.hits) {
              oas[e.id] = e;
            }
            console.log(oas);

            this.oas = oas;
          }

          if (response.internalCode === 401) {
            this.$router.push("/login");
          }
        });
    },

    loadQuestion(id) {
      fetch(this.endpoint + `questions/${this.$route.params.id}`, {
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
            this.question = response.payload;

            this.loadOAS(
              this.question.book[0].area,
              this.question.book[0].level
            );
          }

          if (response.internalCode === 401) {
            this.$router.push("/login");
          }
        });
    },
  },
};
</script>

<style>
</style>
