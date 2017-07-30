<template>
  <div id="control" class="row">
    <div class="col">
      <label class="control-label" for="speed">Speed</label>
      <input class="form-control" name="speed" type="range" v-model.number="speed" min="0" :max="DELAYS.length - 1"/>
      <div class="btn-group">
        <button type="button" class="btn btn-default" @click="$emit('restart')">
          <i class="fa fa-refresh" aria-hidden="true"></i>
        </button>
        <button type="button" class="btn btn-default" v-if="playing" @click="playing = false">
          <i class="fa fa-pause" aria-hidden="true"></i>
        </button>
        <button type="button" class="btn btn-default" v-else @click="playing = true">
          <i class="fa fa-play" aria-hidden="true"></i>
        </button>
        <button type="button" class="btn btn-default" @click="$emit('nextStep')">
          <i class="fa fa-step-forward" aria-hidden="true"></i>
        </button>
      </div>
    </div>
    <transition name="fade">
      <div class="col">
        <div class="row" v-if="mode === 'parser'">
          <div class="col">
            <label class="control-label" for="stateWidthLeft">Left</label>
            <input class="form-control" name="stateWidthLeft" type="number" v-model.number="stateWidthLeft" min="1" max="6"/>
          </div>
          <div class="col">
            <label class="control-label" for="stateWidthRight">Right</label>
            <input class="form-control" type="number" name="stateWidthRight" v-model.number="stateWidthRight" min="1" max="6"/>
          </div>
        </div>
      </div>
    </transition>
    <transition name="fade">
      <div class="col btn-column">
        <button v-if="mode === 'parser'" class="btn btn-primary" @click="$emit('stopAnimation')">On to name generation!</button>
        <button v-else class="btn btn-default" @click="$emit('clearData')">Back to the parser!</button>
      </div>
    </transition>

    <label>
      <input type="checkbox" v-model="strictStarts"/>
      <abbr title="Is this how it works?">Strict start</abbr>
    </label>
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
      stateWidthRight: 2,

      strictStarts: true
    }
  },
  computed: {
    settings () {
      const { playing, stateWidthLeft, stateWidthRight, strictStarts } = this.$data

      return {
        playing,
        stateWidthLeft,
        stateWidthRight,
        strictStarts,
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

<style>
  .btn-column {
    display: flex;
    flex-direction: column;
    justify-content: center;
  }
</style>
