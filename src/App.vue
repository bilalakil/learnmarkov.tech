<template>
  <div id="app">
    <transition appear name="fade" mode="out-in">
      <div v-if="data === null">Loading...</div>
      <markovChain v-else :data="data"></markovChain>
    </transition>
  </div>
</template>

<script>
import MarkovChain from './components/MarkovChain'

export default {
  name: 'app',
  data () {
    return {
      data: null
    }
  },
  components: {
    MarkovChain
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
