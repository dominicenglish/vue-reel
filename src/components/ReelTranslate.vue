<template>
  <div>
    <div class="info">
      <div>{{ isDragging.toString() }}</div>
      <div>{{ this.activeImageIndex }}</div>
      <div>{{ visibleIndexes }}</div>
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
      <template v-for="(image, index) in images">
        <img
          :src="image"
          draggable="false"
          :key="index"
          :class="{
            visible: visibleIndexes.includes(index),
            active: index === activeImageIndex,
          }"
        />
      </template>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ReelTranslate',
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
      // How many images on either side of the active one should be made visible
      visibleDistance: 20,
    }
  },
  computed: {
    // How many horizontal pixels you have to drag to step one frame
    frameChangeDistance() {
      if (!document.body.clientWidth || !this.images.length) return 0
      return (
        (document.body.clientWidth / this.images.length) *
        this.rotationScalingFactor
      )
    },
    totalFrames() {
      return this.images.length
    },
    visibleIndexes() {
      if (!this.totalFrames) return
      const visibleIndexes = []
      for (
        let i = this.activeImageIndex - this.visibleDistance;
        i <= this.activeImageIndex + this.visibleDistance;
        i++
      ) {
        visibleIndexes.push(i < 0 ? i + this.totalFrames : i)
      }
      return visibleIndexes
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
    shouldBeVisible(index) {
      if (index === 179) debugger
      const active = this.activeImageIndex
      const current = index
      const distance = 1
      const visibleIndexes = []
      for (let i = active - distance; i < active + distance; i++) {
        visibleIndexes.push(i ? i : i + this.totalFrames)
      }
      const lowerBound = active - 1
      const upperBound = active + 1
      if (lowerBound <= current && current <= upperBound) {
        return true
      }
      return false
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
  // visibility: hidden
  display: none
  transform: translateX(-100vw)
.visible
  // visibility: visible
  display: block
.active
  /* opacity: 1 */
  // visibility: visible
  transform: translateX(0)
</style>
