<template>
  <InputField msg="Cross-Docking Assignment Problem (CDAP) Viewer" />
</template>

<script>
import InputField from './components/InputField.vue'

export default {
  name: 'App',
  components: {
    InputField
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
