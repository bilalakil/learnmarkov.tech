<template>
  <div id="control">
    <label>
      Speed:
      <input type="range" v-model.number="speed" min="0" :max="DELAYS.length - 1"/>
    </label>

    <button @click="$emit('restart')">Restart</button>
    <button v-if="playing" @click="playing = false">Pause</button>
    <button v-else @click="playing = true">Play</button>
    <button @click="$emit('nextStep')">Step</button>

    <transition name="fade">
      <div v-if="mode === 'parser'">
        <label>
          State width (left):
          <input type="number" v-model.number="stateWidthLeft" min="1" max="6"/>
        </label>
        <label>
          State width (right):
          <input type="number" v-model.number="stateWidthRight" min="1" max="6"/>
        </label>
        <button @click="$emit('stopAnimation')">On to name generation!</button>
      </div>
      <div v-else>
        <button @click="$emit('clearData')">Back to the parser!</button>
      </div>
    </transition>
  </div>
</template>

<script>
export default {
  name: 'controller',
  props: {
    mode: String
  },
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
