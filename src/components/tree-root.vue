<template lang="pug">
.tree-root(ref="root")
  span.tree-root__title {{ title }}
  span.tree-root__current-path(v-if="currentPathToRender") {{ currentPathToRender }}
  tree-node(
    v-for="(item, index) in currentList"
    :key="item.name"
    :list="currentList"
    :chosen-item="chosenItem"
    :paddingTop="paddingTop"
    :root="$refs.root"
    @pick-up="setPickedUpItemHandler"
  )
</template>

<script>
import TreeNode from './tree-node.vue';

export default {
  name: 'TreeRoot',

  components: {
    TreeNode
  },

  data () {
    return {
      chosenItem: null,
      currentPath: '',
      paddingTop: 10,
      title: 'Структурное дерево файлов'
    }
  },

  props: {
    currentList: {
      type: [Object, Array],
      default: () => ({})
    }
  },

  computed: {
    currentPathToRender () {
      return this.currentPath ? `Выбранное значение: ${this.currentPath}` : '';
    }
  },

  methods: {
    setPickedUpItemHandler ({ item, currentPath }) {
      this.chosenItem = item;
      this.currentPath = currentPath;
    }
  }
}
</script>

<style>
body {
  background-color: #0a1231;
}

.tree-root__title {
  margin-bottom: 30px;
  font-size: 26px;
  font-weight: bold;
}

.tree-root__current-path {
  padding-left: 20px;
}

</style>
