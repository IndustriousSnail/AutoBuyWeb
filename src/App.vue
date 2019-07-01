<template>
  <div id="app">
    <!--<img src="./assets/logo.png">-->
    <snail-main></snail-main>
  </div>
</template>

<script>
import SnailMain from "./component/main";
export default {
  components: {SnailMain},
  methods: {
    startHacking () {
      this.$notify({
        title: 'It works!',
        type: 'success',
        message: 'We\'ve laid the ground work for you. It\'s time for you to build something epic!',
        duration: 5000
      })
    }
  }
}
</script>

<style>
#app {
  font-family: Helvetica, sans-serif;
  text-align: center;
}
</style>
