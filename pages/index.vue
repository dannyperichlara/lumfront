<template>
  <div class="container">
    <div>
      <Logo />
      <h1 class="title">
        LUM
      </h1>
    </div>
  </div>
</template>

<script>
export default {
  mounted() {
    if (localStorage.lumkey) {
      if (localStorage.rol === 'teacher') {
        this.$router.push("dashboard")
      } else {
        this.$router.push("estudiante")
      }
    } else {
      this.$router.push("login")
    }
  }
}
</script>

<style>

</style>
