<template>
  <div class="container">
    <div>
      <Logo />
      <h1 class="title">
        SI ERES PROFESOR
      </h1>
      <hr>
      <form>
        <label>Email</label>
        <input v-model="username" id="username" name="username">
        <br>
        <label>Password</label>
        <input v-model="password" type="password">
        <br>
        <b-button v-on:click="login()">ENTRAR</b-button>
      </form>
    </div>
    <hr>
    <div>
      <!-- <Logo /> -->
      <h1 class="title">
        SI ERES ALUMNO
      </h1>
      <hr>
      <form>
        <label>Email</label>
        <input v-model="student.rut" id="rut" name="rut">
        <br>
        <label>Password</label>
        <input v-model="student.password" type="password">
        <br>
        <b-button v-on:click="loginStudent()">ENTRAR</b-button>
      </form>
    </div>
  </div>
</template>

<script>
import config from '~/config.js'
export default {
  data() {
    return {
      username: null,
      password: null,
      endpoint: config.endpoint,
      student: {
        rut: null,
        password: null
      }
    }
  },

  mounted() {
    
  },

  methods: {
    loginStudent() {
      let data = {
        rut: this.student.rut,
        password: this.student.password
      }

      console.log('Logging in')
      fetch(this.endpoint + 'loginStudent', {
        method: 'POST', // or 'PUT'
        mode: 'cors',
        body: JSON.stringify({
          rut: data.rut,
          password: data.password,
        }), // data can be `string` or {object}!
        headers:{
          'Content-Type': 'application/json'
        },

      }).then(res => res.json())
        .catch(error => console.error('Error:', error))
        .then(response => {
          console.log(response)
          if (response.internalCode === 200) {
            localStorage.lumkey = response.payload.token
            this.$router.push('estudiante')
          }
        });
    },

    login() {
      console.log('Logging in')
      fetch(this.endpoint + 'login', {
        method: 'POST', // or 'PUT'
        mode: 'cors',
        body: JSON.stringify({
          username: this.username,
          password: this.password,
        }), // data can be `string` or {object}!
        headers:{
          'Content-Type': 'application/json'
        },

      }).then(res => res.json())
        .catch(error => console.error('Error:', error))
        .then(response => {
          if (response.internalCode === 200) {
            localStorage.lumkey = response.payload.token
            this.$router.push('dashboard')
          }
        });
      }
  }
}
</script>

<style>
/* .container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: left;
  align-items: center;
  text-align: center;
}

.title {
  font-family:
    'Quicksand',
    'Source Sans Pro',
    -apple-system,
    BlinkMacSystemFont,
    'Segoe UI',
    Roboto,
    'Helvetica Neue',
    Arial,
    sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
} */
</style>
