<template>
  <div>
    <div class="c-off-canvas-overlay" :style="{backgroundColor: overlayBackground}" transition="off-canvas-overlay" @click.stop="close" v-if="isOpen"></div>
    <div class="c-off-canvas" :transition="transitionClass" @transitionend="onTransitionEnd" v-show="isOpen" :style="{maxWidth: offCanvasMaxWidth}">
      <div class="c-off-canvas__content" :class="[class]">
        <slot></slot>
      </div>
    </div>
  </div>
</template>

<script>
  import {eventBus} from '../helpers'
  import eve from 'dom-events'

  export default {
    props: [
      {
        name: 'overlayBackground',
        type: String,
        default: 'rgba(0, 0, 0, .5)'
      }, {
        name: 'class',
        type: String,
        default: false
      }, {
        name: 'align',
        type: String,
        default: 'left'
      }, {
        name: 'width',
        type: Number,
        default: 300
      }, {
        name: 'ref',
        type: String,
        required: true
      }, {
        name: 'wrapRef',
        type: String,
        default: false
      }
    ],
    data () {
      return {
        isOpen: false
      }
    },
    computed: {
      transitionClass () {
        return `off-canvas-${this.align}`
      },
      offCanvasMaxWidth () {
        return `${this.width}px`
      }
    },
    ready () {
      eventBus.on('toggle:off-canvas', this.onToggle)
      eve.on(document, 'keydown', this.onKeyDown)
    },
    beforeDestroy () {
      eventBus.removeListener('toggle:off-canvas', this.onToggle)
      eve.off(document, 'keydown', this.onKeyDown)
    },
    methods: {
      onKeyDown (event) {
        // check if keydown is 'esc'
        if (this.isOpen && event.which === 27) {
          event.stopPropagation()
          this.close()
        }
      },
      onToggle (offCanvasRef) {
        if (this.ref === offCanvasRef) {
          if (this.isOpen) {
            this.close()
          } else {
            this.open()
          }
        }
      },
      close () {
        eventBus.emit('close:off-canvas-wrap', this.wrapRef)
        this.isOpen = false
      },
      open () {
        eventBus.emit('open:off-canvas-wrap', {
          offCanvasWidth: this.width,
          offCanvasAlign: this.align
        }, this.wrapRef)
        this.isOpen = true
        this.$emit('opened')
      },
      onTransitionEnd () {
        if (!this.isOpen) {
          this.$emit('closed')
        }
      }
    }
  }
</script>