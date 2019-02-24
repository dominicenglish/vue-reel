<script>
export default {
  inject: ['provider'],
  props: {
    image: String,
  },
  data() {
    return {
      loadedImages: [],
      loadedImageCount: 0,
      images: {},
      imageWidth: 0,
      imageHeight: 0,
      imageTop: 0,
      imageLeft: 0,
    }
  },
  computed: {},
  watch: {
    provider: {
      immediate: true,
      handler() {
        this.init()
      },
    },
    image: {
      immediate: true,
      handler(url) {
        this.loadImage(url)
      },
    },
    'provider.canvasWidth': {
      immediate: true,
      handler() {
        this.$forceUpdate()
      },
    },
    'provider.canvasHeight': {
      immediate: true,
      handler() {
        this.$forceUpdate()
      },
    },
  },
  methods: {
    init() {
      this.loadImage(this.image)
    },
    loadImage(url) {
      // If image already loaded
      if (this.images[url]) return
      // const setImageData = image => this.$set(this.images, url, image)
      const image = new Image()
      image.src = url
      const setImageData = this.setImageData.bind(this)
      image.onload = function() {
        setImageData(url, this)
      }
    },
    setImageData(url, image) {
      this.$set(this.images, url, image)

      if (
        this.imageHeight &&
        this.imageWidth &&
        this.imageLeft &&
        this.imageTop
      )
        return
      const context = this.provider.context
      const canvasWidth = context.canvas.width
      const canvasHeight = context.canvas.height
      // Height should be used to used to cover image
      if (image.width / canvasWidth > image.height / canvasHeight) {
        this.imageHeight = canvasHeight
        this.imageWidth = (canvasHeight / image.height) * image.width
        this.imageLeft = -Math.ceil((this.imageWidth - canvasWidth) / 2)
      } else {
        this.imageWidth = canvasWidth
        this.imageHeight = (canvasWidth / image.width) * image.height
        this.imageTop = -Math.ceil((this.imageHeight - canvasHeight) / 2)
      }
    },
  },
  render() {
    if (!this.provider.context) return
    if (!this.images[this.image]) return

    const image = this.images[this.image]
    const context = this.provider.context
    context.imageSmoothingEnabled = false
    context.drawImage(
      image,
      0,
      0,
      image.naturalWidth,
      image.naturalHeight,
      this.imageLeft,
      this.imageTop,
      this.imageWidth,
      this.imageHeight
    )
    // context.drawImage(image, 0, 0, image.width, image.height, 0, 0, 500, 500)
  },
  mounted() {
    // if (!this.provider.context) throw new Error('No canvas context')
  },
}
</script>
