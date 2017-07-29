<template>
  <div id="control">
    <label>
      Speed:
      <input type="range" v-model.number="speed" min="0" :max="DELAYS.length - 1"/>
    </label>
    <label>
      State width (left):
      <input type="number" v-model.number="stateWidthLeft"/>
    </label>
    <label>
      State width (right):
      <input type="number" v-model.number="stateWidthRight"/>
    </label>
    <button @click="$emit('restart')">Restart</button>
    <button v-if="playing" @click="playing = false">Pause</button>
    <button v-else @click="playing = true">Play</button>
    <button @click="$emit('nextStep')">Step</button>
    <button @click="$emit('stopAnimation')">On to name generation!</button>
  </div>
</template>

<script>
export default {
  name: 'controller',
  data () {
    return {
      speed: 1,
      DELAYS: [2000, 1000, 500, 250, 0],
      playing: true,
      stateWidthLeft: 2,
      stateWidthRight: 2
    }
  },
  computed: {
    settings () {
      const { playing, stateWidthLeft, stateWidthRight } = this.$data

      return {
        playing,
        stateWidthLeft,
        stateWidthRight,
        delay: this.DELAYS[this.speed]
      }
    }
  },
  watch: {
    settings: {
      immediate: true,
      handler (val) {
        this.$emit('settings', val)
      }
    }
  }
}
</script>
