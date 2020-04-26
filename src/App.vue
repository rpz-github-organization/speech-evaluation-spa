<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png" />
    <Evaluation />
  </div>
</template>

<script>
import Evaluation from './views/Evaluation.vue';

export default {
  name: 'App',
  components: {
    Evaluation
  }
};
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;

  img {
    width: 40px;
  }
}
</style>
