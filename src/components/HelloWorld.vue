<template>
  <div class="hello">
    <reel v-if="images" :images="images" />
  </div>
</template>

<script>
import Reel from '@/components/Reel.vue'
export default {
  name: 'HelloWorld',
  props: {
    msg: String,
  },
  components: { Reel },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="stylus">
.hello
  width: 100vw
  height: 100vh
</style>
