<template>
  <div>
    <div class="info">
      <div>{{ isDragging.toString() }}</div>
      <div>{{ this.activeImageIndex }}</div>
      <button @click="startRotation">Rotate</button>
      <button @click="stopRotation">Stop</button>
    </div>
    <div class="reel">
      <canvas
        ref="reel-canvas"
        class="reel-canvas"
        :width="provider.canvasWidth"
        :height="provider.canvasHeight"
        @touchstart.prevent="startDrag"
        @touchend.prevent="stopDrag"
        @touchmove.prevent="reel"
        @mousedown.prevent="startDrag"
        @mouseup.prevent="stopDrag"
        @mouseleave.prevent="stopDrag"
        @mouseout.prevent="stopDrag"
        @mousemove.prevent="reel"
      />
      <slot :activeImage="images[activeImageIndex]"></slot>
    </div>
  </div>
</template>

<script>
import debounce from 'lodash/debounce'
export default {
  name: 'Reel',
  props: {
    images: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      isDragging: false,
      activeImageIndex: 0,
      dragStartPosition: 0,
      dragStartImageIndex: 0,
      rotationScalingFactor: 1,
      animationTimeoutID: undefined,
      provider: {
        context: null,
        activeImageIndex: 0,
        canvasWidth: window.innerWidth,
        canvasHeight: window.innerHeight,
      },
    }
  },
  provide() {
    return {
      provider: this.provider,
    }
  },
  computed: {
    frameChangeDistance() {
      return (
        (document.body.clientWidth / this.images.length) *
        this.rotationScalingFactor
      )
    },
  },
  watch: {
    activeImageIndex() {
      this.provider.activeImageIndex = this.activeImageIndex
    },
  },
  methods: {
    noop() {},
    startRotation() {
      const tick = () => {
        this.stepFrame(1)
        this.animationTimeoutID = setTimeout(tick, 50)
      }
      this.animationTimeoutID = setTimeout(tick, 50)
    },
    stopRotation() {
      if (this.animationTimeoutID) {
        clearTimeout(this.animationTimeoutID)
      }
    },
    stepFrame(incrementBy) {
      this.activeImageIndex =
        (this.images.length + this.activeImageIndex + incrementBy) % 180
    },
    startDrag(event) {
      this.isDragging = true
      this.dragStartPosition = event.clientX || event.touches[0].clientX
      this.dragStartImageIndex = this.activeImageIndex
    },
    stopDrag(event) {
      this.isDragging = false
    },
    reel(event) {
      if (this.isDragging) {
        // Total pixel distance travelled
        const distance =
          this.dragStartPosition - (event.clientX || event.touches[0].clientX)
        // Container size divided by number of images to find distance needed to
        // travel to change image (assuming full screen drag is full cycle)
        const sectionSize = document.body.clientWidth / 180
        // The number of frames to increment
        let framesToChange = distance / this.frameChangeDistance
        // Find if we are moving backwards or forwards through the reel
        let direction = Math.sign(framesToChange)
        // Flooring -0.5 gives you 1 not 0
        let absFramesToChange = direction * Math.floor(Math.abs(framesToChange))
        // Find the new image to load
        this.activeImageIndex =
          (this.images.length + this.dragStartImageIndex + absFramesToChange) %
          180
      }
    },
    sizeCanvasToWindow: debounce(function sizeCanvasToWindow(canvas) {
      this.provider.canvasWidth = window.innerWidth
      this.provider.canvasHeight = window.innerHeight
      this.$forceUpdate()
    }, 500),
  },
  created() {},
  mounted() {
    const canvas = this.$refs['reel-canvas']
    this.provider.context = canvas.getContext('2d')
    window.addEventListener('resize', () => this.sizeCanvasToWindow(canvas))
  },
  beforeDestroy() {
    this.stopRotation()
  },
}
</script>

<style lang="stylus" scoped>
.info
  z-index: 1
  position: fixed
  top: 0
  left: 50%
  display: flex
  div
    width: 50px

.reel
  height: 100%
  width: 100%
  position: relative

.active
  visibility: visible
</style>
