<template>
  <div class="container">
    <div>
      <Logo />
      <h1 class="title">
        LUM
      </h1>
      <hr>
      <form>
        <b-label>Email</b-label>
        <input v-model="username">
        <br>
        <b-label>Password</b-label>
        <input v-model="password" type="password">
        <br>
        <b-button v-on:click="login()">ENTRAR</b-button>
      </form>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      endpoint: 'http://45.58.62.235:4000/v1/'
    }
  },

  mounted() {
    
  },

  methods: {
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
.container {
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
}
</style>
