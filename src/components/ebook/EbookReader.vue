<template>
  <div class="ebook-reader">
    <div id="read"></div>
    <div class="ebook-reader-mask" @click="onMaskClick" @touchmove="move" @touchend="moveEnd"></div>
  </div>
</template>

<script type='text/ecmascript-6'>
import { ebookMixin } from '../../utils/mixin'
import Epub from 'epubjs'
import { getFontFamily, saveFontFamily, getFontSize, saveFontSize, getTheme, saveTheme, getLocation } from '../../utils/localStorage'
import { flatten } from '../../utils/book'
global.ePub = Epub
export default {
  mixins: [ebookMixin],
  methods: {
    move (e) {
      let offsetY = 0
      if (this.firstOffsetY) {
        offsetY = e.changedTouches[0].clientY - this.firstOffsetY
        this.setOffsetY(offsetY)
      } else {
        this.firstOffsetY = e.changedTouches[0].clientY
      }
      e.preventDefault()
      e.stopPropagation()
    },
    moveEnd () {
      this.setOffsetY(0)
      this.firstOffsetY = null
    },
    onMaskClick (e) {
      const offsetX = e.offsetX
      const width = window.innerWidth
      if (offsetX > 0 && offsetX < width * 0.3) {
        this.prevPage()
      } else if (offsetX > 0 && offsetX > width * 0.7) {
        this.nextPage()
      } else {
        this.toggleTitleAndMenu()
      }
    },
    prevPage () {
      if (this.rendition) {
        this.rendition.prev().then(() => {
          this.refreshLocation()
        })
        this.hideTitleAndMenu()
      }
    },
    nextPage () {
      if (this.rendition) {
        this.rendition.next().then(() => {
          this.refreshLocation()
        })
        this.hideTitleAndMenu()
      }
    },
    showTitleAndMenu () {
      if (this.menuVisible) {
        this.setSettingVisible(-1)
        this.setFontFamilyVisible(false)
      }
      this.setMenuVisible(!this.menuVisible)
    },
    initFontFamily () {
      let font = getFontFamily(this.fileName)
      if (!font) {
        saveFontFamily(this.fileName, this.defaultFontFamily)
      } else {
        this.rendition.themes.font(font)
        this.setDefaultFontFamily(font)
      }
    },
    initFontSize () {
      let fontSize = getFontSize(this.fileName)
      if (!fontSize) {
        saveFontSize(this.fileName, this.defaultFontSize)
      } else {
        this.rendition.themes.font(fontSize)
        this.setDefaultFontSize(fontSize)
      }
    },
    initTheme () {
      let defaultTheme = getTheme(this.fileName)
      if (!defaultTheme) {
        defaultTheme = this.themeList[0].name
        saveTheme(this.fileName, defaultTheme)
      }
      this.setDefaultTheme(defaultTheme)
      this.themeList.forEach(theme => {
        this.rendition.themes.register(theme.name, theme.style)
      })
      this.rendition.themes.select(defaultTheme)
    },
    parseBook () {
      this.book.loaded.cover.then(cover => {
        this.book.archive.createUrl(cover).then(url => {
          // console.log(url)
          this.setCover(url)
        })
      })
      this.book.loaded.metadata.then(metadata => {
        this.setMetadata(metadata)
      })
      this.book.loaded.navigation.then(nav => {
        const navItem = flatten(nav.toc)
        function find (item, level = 0) {
          return !item.parent ? level : find(navItem.fileter(parentItem => parentItem.id === item.parent)[0], ++level)
        }
        navItem.forEach(item => {
          item.level = find(item)
        })
        this.setNavigation(navItem)
      })
    },
    initRendition () {
      this.rendition = this.book.renderTo('read', {
        width: innerWidth,
        height: innerHeight,
        method: 'default'
      })
      const location = getLocation(this.fileName)
      this.display(location, () => {
        this.initTheme()
        this.initFontSize()
        this.initFontFamily()
        this.initGlobalStyle()
        this.parseBook()
      })
      this.rendition.hooks.content.register(contents => {
        Promise.all([
          contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/fonts/daysOne.css`),
          contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/fonts/cabin.css`),
          contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/fonts/montserrat.css`),
          contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/fonts/tangerine.css`)
        ]).then(() => {})
      })
    },
    initGesture () {
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
          this.nextPage()
        } else {
          this.showTitleAndMenu()
        }
        event.preventDefault()
        event.stopPropagation()
      })
    },
    initEpub () {
      const url = process.env.VUE_APP_RES_URL + '/epub/' + this.fileName + '.epub'
      this.book = new Epub(url)
      this.setCurrentBook(this.book)
      this.initRendition()
      this.initGesture()
      this.book.ready.then(() => {
        return this.book.locations.generate(750 * (window.innerWidth / 375) * (getFontSize(this.fileName) / 16))
      }).then(location => {
        this.setBookAvailable(true)
        this.refreshLocation()
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
@import '../../assets/styles/global';

.ebook-reader {
  width: 100%;
  height: 100%;
  overflow: hidden;
  .ebook-reader-mask {
    position: absolute;
    top: 0;
    left: 0;
    background: transparent;
    z-index: 150;
    width: 100%;
    height: 100%;
  }
}
</style>
