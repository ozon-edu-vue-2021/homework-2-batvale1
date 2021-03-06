<template lang="pug">
ul.tree-node(:style="currentYOffset")
  li.tree-node__item(
    v-for="(item, index) in list"
    tabindex="0"
    ref="item"
    :key="item.name"
    :class="listInfo[item.name].classes"
    @click.stop="clickHandler(item)"
    @keydown.stop.enter="clickHandler(item)"
    @keydown.stop.prevent.down="keyDownHandler(index)"
    @keydown.stop.prevent.up="keyUpHandler(index)"
  ) {{ listInfo[item.name].title }}
    tree-node.tree-node__list(
      v-if="showChildren(item)"
      tabindex="0"
      :list="item.contents"
      :chosen-item="chosenItem"
      :padding-top="paddingTop"
      :root="root"
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
    },
    root: {
      type: [HTMLElement, undefined],
      required: false
    }
  },

  computed: {
    currentYOffset () {
      return { paddingTop: `${this.paddingTop}px` };
    },

    listInfo () {
      const result = {};

      this.list.forEach(item => {
        result[item.name] = {};
        result[item.name].title = this.getTitle(item);
        result[item.name].classes = this.getClass(item);
      })

      return result;
    }
  },

  methods: {
    // handlers
    keyDownHandler (index) {
      if (!this.chosenItem) return;

      const listLength = this.list.length;
      let indexToSet;

      if (index === listLength - 1) {
        const sibling = this.getSibling(this.$el);

        if (sibling) {
          sibling.focus();
        }

        return;
      } else {
        indexToSet = index + 1;
      }

      const nextHTMLElement = this.$refs.item[indexToSet];

      nextHTMLElement.focus();
    },

    keyUpHandler (index) {
      if (!this.chosenItem) return;

      let indexToSet;

      if (index === 0) {
        this.$el.parentElement.focus();
        return;
      } else {
        indexToSet = index - 1;
      }

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
    },

    // service

    getSibling (element) {
      let sibling = element;

      while (sibling !== this.root) {
        const parentElement = sibling.parentElement;

        sibling = parentElement.nextSibling;

        if (!sibling) {
          sibling = parentElement;
        } else {
          return sibling;
        }
      }

      return null;
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
  border: 1px dashed red;
}
</style>