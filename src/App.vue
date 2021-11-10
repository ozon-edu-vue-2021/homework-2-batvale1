<template lang="pug">
  #app
    TreeRoot(
      :current-list="currentList"
    )
</template>

<script>
import TreeRoot from './components/tree-root.vue';
import list from '../public/static/node_modules.json';

export default {
  name: 'App',

  components: {
    TreeRoot
  },

  computed: {
    currentList () {
      return typeof list === 'object' ? [list] : list;
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  color: white;
  margin-top: 60px;
}

body {
  background-color: #0a1231;
}
</style>
