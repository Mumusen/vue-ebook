<template>
  <div class="eboo-reader">
    <div id="read"></div>
  </div>
</template>

<script type='text/ecmascript-6'>
import { mapGetters } from 'vuex'
import Epub from 'epubjs'
global.ePub = Epub
export default {
  computed: {
    ...mapGetters(['fileName'])
  },
  methods: {
    initEpub () {
      const url = 'http://192.168.0.20:8081/epub/' + this.fileName + '.epub'
      this.book = new Epub(url)
      this.rendition = this.book.renderTo('read', {
        width: innerWidth,
        height: innerHeight,
        method: 'default'
      })
      this.rendition.display()
      this.rendition.on('touchstart', event => {
        console.log(event)
      })
      this.rendition.on('touchend', event => {
        console.log(event)
      })
    }
  },
  mounted () {
    this.$store.dispatch('setFileName', this.$route.params.fileName.split('|').join('/'))
      .then(() => {
        this.initEpub()
      })
    // console.log(`${baseUrl}${fileName}.epub`)
  }
}
</script>

<style lang='scss' rel='tylesheet/scss' scoped>
</style>
