<template>
  <div class="container m-5">
    <div>
      <h1 class="title">Editar Pregunta</h1>
      <!-- <p>(Estás en el libro {{question.book.area}}) de {{question.book.name}}</p> -->
      <p class="small">{{ question }}</p>
      <!-- <p class="small">{{ oas }}</p> -->
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
          v-for="(indicator, j) in question.indicators"
          :key="j"
        >
          <b-row>
            <b-col v-if="oas[indicator]"
              >{{ j + 1 }}. {{ oas[indicator].name }}</b-col
            >
            <b-col class="text-right">
              <b-button
                variant="danger"
                v-on:click="removeIndicatorFromQuestion(indicator)"
                >x</b-button
              >
            </b-col>
          </b-row>
        </b-list-group-item>

        <b-button v-b-modal.indicators
          >Asociar nuevo indicador a la pregunta</b-button
        >
      </b-list-group>

      <b-modal
        id="indicators"
        size="xl"
        title="Objetivos Aprendizaje asociados a la pregunta"
      >
        <div>{{ question.indicators }}</div>
        <b-list-group>
          <b-list-group-item
            v-for="(e, i) in oas"
            :key="i"
            v-if="e"
            :class="
              question.indicators.indexOf(e.id) != -1 ? 'active mb-1' : 'mb-1'
            "
            v-on:click="selectIndicator(e.id)"
          >
            {{ e.name }} Unidades: {{ e.unit }}
          </b-list-group-item>
        </b-list-group>
      </b-modal>

      <b-modal id="newAnswer" title="Agregar nueva respuesta" size="lg" @ok="addNewAnswer()">
        <label>Agrega el texto de la respuesta</label>
        <b-textarea rows="5" v-model="newAnswerText"></b-textarea>
      </b-modal>

      <div class="mt-5 h2"><label>Posibles Respuestas:</label></div>
      <b-list-group v-if="question && question.possible_answers">
        <b-list-group-item
          class="h5"
          v-for="(answer, i) in question.possible_answers"
          :key="i"
        >
          <b-row>
            <b-col class="col-1 text-nowrap"
              ><img height="60" :src="answer.image"
            /></b-col>
            <b-col>{{ answer.text }}</b-col>
            <b-col>{{ answer.indicators }}</b-col>
            <b-col class="text-right">
              <b-button variant="dark">Editar</b-button>
              <b-button variant="danger">x</b-button>
            </b-col>
            <b-col class="text-nowrap">
              <span>Indicadores asociados:</span>
              <b-button
                class="ml-2"
                v-for="(indicator, j) in question.indicators"
                :key="j"
                :variant="
                  answer.indicators.indexOf(indicator) != -1
                    ? 'primary'
                    : 'secondary'
                "
                v-on:click="associate_indicator_to_answer(i, indicator)"
              >
                <span>{{ j + 1 }}</span>
              </b-button>
            </b-col>
          </b-row>
        </b-list-group-item>
        <b-button v-b-modal.newAnswer>Agregar nueva posible respuesta</b-button>
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

      newAnswerText: '',

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
    addNewAnswer() {
      if (!this.newAnswerText) return

      let newAnswer = {
        "image": "",
        "idQuestion": this.question.id,
        "indicators": [],
        "text": this.newAnswerText
      }

      let url = this.endpoint + `questions/${this.question.id}/possibleanswers`;

      fetch(url, {
        method: "POST", // or 'PUT'
        mode: "cors",
        headers: {
          "Content-Type": "application/json",
          "x-lum-token": localStorage.lumkey,
        },
        body: JSON.stringify(newAnswer)
      })
      .then((res) => res.json())
      .catch((error) => console.error("Error:", error))
      .then((response) => {
        if (response.internalCode === 200) {
          this.loadQuestion(this.question.id)
        }

        if (response.internalCode === 401) {
          this.$router.push("/login");
        }
      });
    },

    associate_indicator_to_answer(answerIndex, indicatorId) {
      if (
        this.question.possible_answers[answerIndex].indicators.indexOf(
          indicatorId
        ) !== -1
      ) {
        let filtered = this.question.possible_answers[
          answerIndex
        ].indicators.filter((e) => {
          return e !== indicatorId;
        });

        this.question.possible_answers[answerIndex].indicators = filtered;

        return;
      }

      this.question.possible_answers[answerIndex].indicators.push(indicatorId);
    },

    removeIndicatorFromQuestion(id) {
      let filtered = this.question.indicators.filter((e) => e != id);
      this.question.indicators = filtered;

      this.cleanIndicators();
    },

    cleanIndicators() {
      for (let e of this.question.possible_answers) {
        e.indicators = [];
      }
    },

    selectIndicator(id) {
      console.log(this.question.possible_answers);

      this.cleanIndicators();

      console.log(this.question.possible_answers);
      // for (let e of this.question.possibleAnswers) {
      //   console.log(e)
      // }

      if (this.question.indicators.indexOf(id) == -1) {
        this.question.indicators.push(id);
      } else {
        this.removeIndicatorFromQuestion(id);
      }
    },

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
            let oas = [null, { id: 1, name: "xxx1" }, { id: 2, name: "xxx2" }];

            for (let e of response.payload.hits) {
              oas[e.id] = e;
            }

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

            this.question.indicators = JSON.parse(this.question.indicators);

            for (let e of this.question.possible_answers) {
              console.log(e.indicators);
              e.indicators = JSON.parse(e.indicators).map((e) => parseInt(e));
            }

            console.log(this.question.possible_answers);

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
