<template>
  <div class="container m-5">
    <div>
      <p v-if="question.book"><a :href="'/class/' + question.idBook" >(Volver a {{question.book.area}} de {{question.book.name}})</a></p>
      <h1 class="title">Editar Pregunta</h1>
      <div class="text-right">
        <b-button variant="success" v-on:click="saveQuestion()">
          Guardar
        </b-button>
      </div>
      <b-row class="mt-5" v-if="question">
        <b-col class="col-4">
          <div><label>Imagen de la pregunta (click para cambiar):</label></div>
          <img width="100%" v-on:click="newQuestionImage()" :src="question.image" onerror="this.onerror=null;this.src='/images/photo.png';" ></img>
          <input type="file" id="file-upload" v-show="false" />
        </b-col>
        <b-col>
          <div><label>Texto de la pregunta:</label></div>
          <textarea
            rows="8"
            class="w-100 p-4"
            v-model="question.text"
          ></textarea>
        </b-col>
      </b-row>

      <div class="mt-5 h2">
        <label>Indicadores asociados a la pregunta</label>
      </div>
      <b-list-group v-if="question && question.indicators && oas">
        <b-list-group-item
          class="small"
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
          class=""
          v-for="(answer, i) in question.possible_answers"
          :key="i"
        >
          <b-row>
            <b-col class="col-1 text-nowrap"
              ><img height="60" :src="answer.image" v-on:click="newAnswerImage(i)" onerror="this.onerror=null;this.src='/images/photo.png';"
            /></b-col>
            <b-col class="ml-2"><b-textarea type="text" v-model="answer.text"></b-textarea></b-col>
            <b-col class="text-nowrap">
              <span>Indicadores asociados a la respuesta:</span><br>
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
            <b-col class="text-right">
              <b-button variant="danger" v-on:click="removeAnswer(answer.id)">x</b-button>
            </b-col>

          </b-row>
        </b-list-group-item>
        <b-button v-b-modal.newAnswer>Agregar nueva posible respuesta</b-button>
      </b-list-group>
      <div class="text-right mt-5">
        <b-button variant="success" v-on:click="saveQuestion()">
          Guardar
        </b-button>
      </div>
    </div>
  </div>
</template>

<script>
import config from "~/config.js";

import axios from "axios"

export default {
  data() {
    return {
      endpoint: config.endpoint,

      oas: new Array(800),

      newAnswerText: '',

      question: {
        text: "",
        indicators: "[]",
        book: {},

        possibleAnswers: [],
      },
    };
  },

  created() {
    this.loadQuestion();
  },

  mounted() {
    let file = document.getElementById('file-upload')

    file.addEventListener('change', e=>{
      let image = e.target.files[0]
      let upload_preset = 'ftfspykv'
      let endpoint = 'https://api.cloudinary.com/v1_1/dbxyj0qdg/upload'
      let formData = new FormData()

      formData.append('file', image)
      formData.append('upload_preset', upload_preset)

      //Se intentó realizar este mismo algoritmo con la función fetch pero arroja error 502
      axios({
        url: endpoint,
        method: 'POST',
        // mode: 'no-cors',
        headers: {
          "Content-Type": "application/x-www-form-urlencoded",
        },
        data: formData
      })
      .then((res)=>{
        console.log(res)
        this.question.image = res.data.url
      })
      .catch((error) => console.error("Error:", error))
    })
  },

  methods: {
    newAnswerImage(i) {
      let input = document.createElement('input')
      input.type = "file"

      input.addEventListener('change', e=>{
        let image = e.target.files[0]
        let upload_preset = 'ftfspykv'
        let endpoint = 'https://api.cloudinary.com/v1_1/dbxyj0qdg/upload'
        let formData = new FormData()

        formData.append('file', image)
        formData.append('upload_preset', upload_preset)

        //Se intentó realizar este mismo algoritmo con la función fetch pero arroja error 502
        axios({
          url: endpoint,
          method: 'POST',
          // mode: 'no-cors',
          headers: {
            "Content-Type": "application/x-www-form-urlencoded",
          },
          data: formData
        })
        .then((res)=>{
          console.log(res)
          this.question.possible_answers[i].image = res.data.url
        })
        .catch((error) => console.error("Error:", error))
      })
      
      input.click()
    },

    newQuestionImage() {
      document.getElementById('file-upload').click()
    },

    saveQuestion() {
      let url = this.endpoint + `questions/${this.question.id}`;

      axios({
        url,
        method: "PUT",
        // mode: "cors",
        headers: {
          "Content-Type": "application/json",
          "x-lum-token": localStorage.lumkey,
        },
        data: this.question
      })
      .then((res) => res.json())
      .catch((error) => console.error("Error:", error))
      .then((response) => {
        console.log(response)
      })
    },

    removeAnswer(id) {
      let url = this.endpoint + `possibleanswers/${id}`;

      fetch(url, {
        method: "DELETE", // or 'PUT'
        mode: "cors",
        headers: {
          "Content-Type": "application/json",
          "x-lum-token": localStorage.lumkey,
        }
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
      if (!this.acceptIndicatorsChange()) return

      let filtered = this.question.indicators.filter((e) => e != id);
      this.question.indicators = filtered;

      this.cleanIndicators();
    },

    acceptIndicatorsChange() {
      return confirm('Si cambias los indicadores, deberás asociarlos nuevamente a cada posible respuesta. ¿Estás seguro que deseas realizar esta acción?')
    },

    cleanIndicators() {
       
      for (let e of this.question.possible_answers) {
        e.indicators = [];
      }
    },

    selectIndicator(id) {
      if (!this.acceptIndicatorsChange()) return

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

      let url = this.endpoint + `questions/${this.question.id}`;

      fetch(url, {
        method: "PUT", // or 'PUT'
        mode: "cors",
        headers: {
          "Content-Type": "application/json",
          "x-lum-token": localStorage.lumkey,
        },
        body: JSON.stringify({
          text: this.question.text,
          image: this.question.image,
          indicators: JSON.stringify(this.question.indicators),
          id: this.question.id
        })
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
            console.log(this.question);

            for (let e of this.question.possible_answers) {
              console.log(e.indicators);
              e.indicators = JSON.parse(e.indicators).map((e) => parseInt(e));
            }


            this.loadOAS(
              this.question.book.area,
              this.question.book.level
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
