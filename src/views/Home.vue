<template>
  <div class="home">
    <!-- <Reel v-if="images" :images="images" class="reel" /> -->
    <!-- <ReelTranslate v-if="images" :images="images" class="reel" /> -->
    <ReelCanvas v-if="images" :images="images" class="reel">
      <template v-slot:default="slotProps">
        <ReelImage :image="slotProps.activeImage" />
      </template>
    </ReelCanvas>
  </div>
</template>

<script>
// @ is an alias to /src
import Reel from '@/components/Reel.vue'
import ReelTranslate from '@/components/ReelTranslate.vue'
import ReelCanvas from '@/components/ReelCanvas.vue'
import ReelImage from '@/components/ReelImage.vue'

export default {
  name: 'home',
  components: {
    Reel,
    ReelTranslate,
    ReelCanvas,
    ReelImage,
  },
  data() {
    const images = Array.from({ length: 180 }).map((val, index) => {
      const imageName = (index + 1).toString().padStart(3, '0')
      return `/img/rotation/${imageName}.jpg`
    })
    return {
      images,
    }
  },
}
</script>

<style scoped lang="stylus">
.home
  width: 100vw
  height: 100vh

.reel
  width: 100vw
  height: 100vh
</style>
