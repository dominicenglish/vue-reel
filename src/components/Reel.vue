<template>
  <div>
    <div class="info">
      <div>{{ isDragging.toString() }}</div>
      <div>{{ this.activeImageIndex }}</div>
      <button @click="startRotation">Rotate</button>
      <button @click="stopRotation">Stop</button>
    </div>
    <div
      class="reel"
      @mousedown.prevent.stop="startDrag"
      @mouseup.prevent.stop="stopDrag"
      @mouseleave.prevent.stop="stopDrag"
      @mousemove.prevent.stop="reel"
    >
      <img
        v-for="(image, index) in images"
        :src="image"
        :class="{ active: index === activeImageIndex }"
        draggable="false"
      />
    </div>
  </div>
</template>

<script>
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
      mouseDownPosition: 0,
      mouseDownImageIndex: 0,
      rotationScalingFactor: 1,
      animationTimeoutID: undefined,
    }
  },
  computed: {
    // How many horizontal pixels you have to drag to step one frame
    frameChangeDistance() {
      return (
        (document.body.clientWidth / this.images.length) *
        this.rotationScalingFactor
      )
    },
  },
  methods: {
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
      this.mouseDownPosition = event.clientX
      this.mouseDownImageIndex = this.activeImageIndex
    },
    stopDrag(event) {
      this.isDragging = false
    },
    reel(event) {
      if (this.isDragging) {
        // Total pixel distance travelled
        const distance = this.mouseDownPosition - event.clientX
        // Container size divided by number of images to find distance needed to
        // travel to change image (assuming full screen drag is full cycle)
        const sectionSize = document.body.clientWidth / 180
        // The number of frames to increment
        const framesToChange = Math.floor(distance / this.frameChangeDistance)

        this.activeImageIndex =
          (this.images.length + this.mouseDownImageIndex + framesToChange) % 180
      }
    },
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

img
  height: 100%
  width: 100%
  object-fit: cover
  object-position: center
  position: absolute
  top: 0
  left: 0
  /* opacity: 0 */
  visibility: hidden

.active
  /* opacity: 1 */
  visibility: visible
</style>
