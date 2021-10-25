<template lang="pug">
ul.tree-node(:style="currentYOffset")
  li.tree-node__item(
    v-for="(item, index) in list"
    tabindex="0"
    ref="item"
    :key="index"
    :class="getClass(item)"
    @click.stop="clickHandler(item)"
    @keydown.stop.prevent.down="keyDownHandler(index)"
    @keydown.stop.prevent.up="keyUpHandler(index)"
  ) {{ getTitle(item) }}
    tree-node.tree-node__list(
      v-if="showChildren(item)"
      :list="item.contents"
      :chosen-item="chosenItem"
      :padding-top="paddingTop"
      @pick-up="(payload) => pickUpItemHandler(item, payload)"
    )
</template>

<script>
export default {
  name: 'TreeNode',

  data () {
    return {
      visibleItems: {}
    }
  },

  props: {
    paddingTop: {
      type: Number,
      default: 0
    },
    list: {
      type: Array,
      default: () => ([])
    },
    chosenItem: {
      type: Object,
      default: () => ({})
    }
  },

  computed: {
    currentYOffset () {
      return { paddingTop: `${this.paddingTop}px` };
    }
  },

  methods: {
    // handlers
    keyDownHandler (index) {
      if (!this.chosenItem) return;

      const listLength = this.list.length;
      let indexToSet;

      if (index === listLength - 1) {
        indexToSet = 0;
      } else {
        indexToSet = index + 1;
      }

      this.pickUpItemHandler(this.list[indexToSet]);

      const nextHTMLElement = this.$refs.item[indexToSet];

      nextHTMLElement.focus();
    },

    keyUpHandler (index) {
      if (!this.chosenItem) return;

      const listLength = this.list.length;
      let indexToSet;

      if (index === 0) {
        indexToSet = listLength - 1;
      } else {
        indexToSet = index - 1;
      }

      this.pickUpItemHandler(this.list[indexToSet]);

      const nextHTMLElement = this.$refs.item[indexToSet];

      nextHTMLElement.focus();
    },

    clickHandler (item) {
      this.pickUpItemHandler(item);
      this.toggleItemVisibility(item);
    },

    pickUpItemHandler (item, payload = null) {
      const currentPath = payload ? `/${item.name}${payload.currentPath}` : `/${item.name}`;
      const currentItem = payload ? payload.item : item;

      this.$emit('pick-up', { item: currentItem, currentPath });
    },

    toggleItemVisibility (item) {
      if (this.visibleItems[item.name]) {
        this.$delete(this.visibleItems, [item.name]);
      } else {
        this.$set(this.visibleItems, [item.name], true);
      }
    },

    // conditional rendering
    showChildren (item) {
      return this.visibleItems[item.name] && item.contents && item.contents.length;
    },

    getTitle (item) {
      const isFolder = item.type === 'directory';

      if (!isFolder) return item.name;

      return this.visibleItems[item.name] ? `${item.name} -` : `${item.name} +`;
    },

    // styles

    getClass (item) {
      let typeClass = '';

      switch (item.type) {
        case 'directory': typeClass = 'tree-node__item--dir'; break;
        case 'file': typeClass = 'tree-node__item--file'; break;
        case 'link': typeClass = 'tree-node__item--link'; break;
        default: typeClass = '';
      }

      const isActiveClass = this.chosenItem === item;

      return [
        typeClass,
        { 'tree-node__item--is-active': isActiveClass }
      ]
    }
  }
};
</script>

<style scoped>
.tree-node {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  list-style-type: none;
  padding: 0;
}

.tree-node__list {
  padding-left: 20px;
}

.tree-node__item {
  padding: 5px 15px;
  cursor: pointer;
  border: 1px solid transparent;
}

.tree-node__item--dir {
  background-color: #2c3e50;
}

.tree-node__item--file {
  background-color: #64421a;
}

.tree-node__item--link {
  background-color: #2b0350;
}

.tree-node__item--is-active {
  border: 1px dashed blue;
}

.tree-node__item:focus {
  outline: none;
}
</style>