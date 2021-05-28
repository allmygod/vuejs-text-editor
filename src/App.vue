<template>
  <div>
    <h3>VueJS Text Editor</h3>
    <editor :width="792" :height="612" />
  </div>
</template>

<script>
import Editor from './components/Editor'

export default {
  name: 'App',
  components: {
    Editor
  }
}
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #222;
  margin: 16px;
}
</style>
