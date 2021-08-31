<template>
  <div>
    <b-navbar toggleable="lg" type="dark" variant="info">
      <b-navbar-brand href="#">LUM</b-navbar-brand>
      <b-navbar-nav class="ml-auto" v-on:click="logout()">Salir</b-navbar-nav>
    </b-navbar>

    <Nuxt />
  </div>
</template>

<script>
export default {
  methods: {
    logout() {
      localStorage.removeItem('lumkey')
      location.reload()
    }
  }
}
</script>

<style>

</style>
