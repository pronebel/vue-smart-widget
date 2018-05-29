<template>
  <grid-layout
    v-bind="layoutAttribute"
    @layout-updated="layoutUpdatedEvent"
  >
    <grid-item
      v-for="item in testLayout"
      :key="item.i"
      v-bind="item"
      dragIgnoreFrom=".widget-body"
      @move="moveEvent"
      @resize="resizeEvent"
      @moved="movedEvent"
      @resized="resizedEvent"
    >
      <slot :name="item.i"></slot>
    </grid-item>
  </grid-layout>
</template>

<script>
import bus from './bus'

import { GridLayout, GridItem } from 'vue-grid-layout'

export default {
  name: 'SmartWidgetGroup',
  components: {
    GridLayout,
    GridItem
  },
  provide () {
    return {
      layout: this.layout
    }
  },
  props: {
    layout: {
      type: Array,
      required: true
    },
    draggable: {
      type: Boolean,
      default: true
    },
    resizable: {
      type: Boolean,
      default: true
    },
    colNum: {
      type: Number,
      default: 12
    },
    margin: {
      type: Array,
      default: () => [10, 10]
    },
    rowHeight: {
      type: Number,
      default: 48
    }
  },
  data () {
    return {
      layoutAttribute: {
        autoSize: true,
        layout: this.layout,
        colNum: this.colNum,
        rowHeight: this.rowHeight,
        margin: this.margin,
        isDraggable: this.draggable,
        isResizable: this.resizable,
        isMirrored: false,
        verticalCompact: true,
        useCssTransforms: true
      },
      testLayout: this.layout,
      oldItemH: ''
    }
  },
  methods: {
    layoutUpdatedEvent (newLayout) {
      this.$emit('layout-updated', newLayout)
    },
    moveEvent (i, newX, newY) {
      this.$emit('move', { i, newX, newY })
    },
    resizeEvent (i, newH, newW, newHPx, newWPx) {
      this.$emit('resize', { i, newH, newW, newHPx, newWPx })
    },
    movedEvent (i, newX, newY) {
      this.$emit('layout-updated', { i, newX, newY })
    },
    resizedEvent (i, newH, newW, newHPx, newWPx) {
      this.$emit('layout-updated', { i, newH, newW, newHPx, newWPx })
    }
  },
  created () {
    bus.$on('on-collapsed', ({ i, isCollapsed }) => {
      const itemIndex = this.layout.findIndex(v => v.i === i)
      if (isCollapsed) {
        this.$nextTick(_ => {
          this.layout[itemIndex].oldH = this.layout[itemIndex].h
          this.testLayout[itemIndex].h = 1
        })
      } else {
        this.$nextTick(_ => {
          this.testLayout[itemIndex].h = this.layout[itemIndex].oldH
        })
      }
    })
  }
}
</script>

<style lang="less">
.vue-grid-layout {
  background: transparent;
  // .vue-grid-item {}
  .smartwidget {
    height: inherit;
    width: inherit;
  }
}
</style>
