<template>
  <div class="eboo-reader">
    <div id="read"></div>
  </div>
</template>

<script type='text/ecmascript-6'>
import { ebookMixin } from '../../utils/mixin'
import Epub from 'epubjs'
global.ePub = Epub
export default {
  mixins: [ebookMixin],
  methods: {
    prevPage () {
      if (this.rendition) {
        this.rendition.prev()
        this.hideTitleAndMenu()
      }
    },
    nextPate () {
      if (this.rendition) {
        this.rendition.next()
        this.hideTitleAndMenu()
      }
    },
    showTitleAndMenu () {
      this.setMenuVisible(!this.menuVisible)
    },
    hideTitleAndMenu () {
      this.$store.dispatch('setMenuVisible', !this.menuVisible)
      this.setMenuVisible(false)
    },
    initEpub () {
      const baseUrl = 'http://192.168.0.41:8081/epub/' + this.fileName + '.epub'
      this.book = new Epub(baseUrl)
      this.rendition = this.book.renderTo('read', {
        width: innerWidth,
        height: innerHeight,
        method: 'default'
      })
      this.rendition.display()
      this.rendition.on('touchstart', event => {
        // console.log(event)
        this.touchStartX = event.changedTouches[0].clientX
        this.touchStartTime = event.timeStamp
      })
      this.rendition.on('touchend', event => {
        const offsetX = event.changedTouches[0].clientX - this.touchStartX
        const tiem = event.timeStamp - this.touchStartTime
        if (tiem < 500 && offsetX > 40) {
          this.prevPage()
        } else if (tiem < 500 && offsetX < -40) {
          this.nextPate()
        } else {
          this.showTitleAndMenu()
        }
        event.preventDefault()
        event.stopPropagation()
      })
    }
  },
  mounted () {
    this.setFileName(this.$route.params.fileName.split('|').join('/')).then(() => {
      this.initEpub()
    })
  }
}
</script>

<style lang='scss' rel='tylesheet/scss' scoped>
</style>
