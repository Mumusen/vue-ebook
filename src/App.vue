<template>
  <div id="app">
    <span class="text">ABCDEFG</span>
    <router-view/>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
const getters = {
  a: () => 1,
  b: () => 2
}
function fn (keys) {
  const data = {}
  keys.forEach(key => {
    if (getters.hasOwnProperty(key)) {
      data[key] = getters[key]
    }
  })
  return data
}
export default {
  computed: {
    ...mapGetters(['test']),
    ...fn(['a', 'b'])
  },
  mounted () {
    // this.$store.dispatch('setTest', 8).then(() => {
    //   console.log(this.test)
    // })
    console.log(this.a, this.b)
  }
}
document.addEventListener('DOMContentLoaded', () => {
  const html = document.querySelector('html')
  let fontSize = window.innerWidth / 10
  fontSize = fontSize > 50 ? 50 : fontSize
  html.style.fontSize = fontSize + 'px'
})
</script>

<style scoped>
.text {
  font-family: 'Days One';
  font-size: 1rem;
  color: orange;
}
</style>
