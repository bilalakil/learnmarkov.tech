<template>
  <div id="app">
    <transition appear name="fade" mode="out-in">
      <div v-if="data === null">Loading...</div>
      <trainer v-else :data="data"></trainer>
    </transition>
  </div>
</template>

<script>
import Trainer from './components/Trainer'

export default {
  name: 'app',
  data () {
    return {
      data: null
    }
  },
  components: {
    Trainer
  },
  methods: {
    loadNames (path) {
      const self = this

      const err = () => { console.error('Failed to download data.') }

      const req = new XMLHttpRequest()
      req.open('GET', path)
      req.onload = () => {
        if (req.status === 200) {
          self.data = req.response.split('\n')
        } else {
          err()
        }
      }
      req.onerror = err
      req.send()
    }
  },
  mounted () {
    this.loadNames('/static/companynames.txt')
  }
}
</script>

<style scoped>
.fade-enter-active, .fade-leave-active {
  transition: opacity 1s
}
.fade-enter, .fade-leave-to {
  opacity: 0
}
</style>
