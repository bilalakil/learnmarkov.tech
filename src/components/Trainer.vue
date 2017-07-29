<template>
  <div :style="{ '--delay': delay }">
    <transition appear name="fade">
      <div v-if="currentName" class="current-name">
        <span v-for="(letter, i) in currentName"
          class="letter"
          :class="{ 'in-state': indexInState(i), 'not-in-state': !indexInState(i) }"
        >{{ letter === ' ' ? '_' : letter }}</span>
      </div>
    </transition>
    <div id="control">
      <label>
        Speed:
        <input type="range" v-model.number="speed" min="0" v-bind:max="10"/>
      </label>
      <label>
        State width (left):
        <input type="number" v-model.number="stateWidthLeft"/>
      </label>
      <label>
        State width (right):
        <input type="number" v-model.number="stateWidthRight"/>
      </label>
      <button @click="restart">Restart</button>
    </div>
  </div>
</template>

<script>
const animSpeed = 1000

export default {
  name: 'trainer',
  props: {
    data: Array
  },
  data () {
    return {
      speed: 1,
      nextStep: this.nextName,

      stateWidthLeft: 2,
      stateWidthRight: 2,

      stateTransitions: {},

      nameCursor: -1,
      stateCursor: -1
    }
  },
  computed: {
    delay () {
      return animSpeed / this.speed
    },
    currentName () {
      if (this.nameCursor !== -1 && this.nameCursor < this.data.length) {
        return this.data[this.nameCursor]
      }

      return
    }
  },
  methods: {
    next (delayMult) {
      delayMult = typeof delayMult === 'undefined'
        ? 1
        : delayMult

      setTimeout(this.nextStep, this.delay * delayMult)
    },

    nextName () {
      this.nameCursor += 1

      if (this.nameCursor !== this.data.length) {
        this.nextStep = this.nextState
        this.next()
      }
    },
    nextState () {
      this.stateCursor += 1

      if (this.stateCursor !== this.currentName.length - this.stateWidthLeft) {
        this.nextStep = this.nextState
        this.next()
      }
    },

    restart () {
      this.speed = 1
      this.nextStep = this.nextName

      this.stateTransitions = {}

      this.nameCursor = -1
      this.stateCursor = -1

      this.next(1.5)
    },

    indexInState (i) {
      return this.stateCursor !== -1 &&
        i >= this.stateCursor &&
        i < this.stateCursor + this.stateWidthLeft
    }
  },
  mounted () {
    this.next(1.5)
  }
}
</script>

<style scoped>
.fade-enter-active, .fade-leave-active {
  transition: opacity 1s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}

.current-name .letter {
  display: inline-block;

  transition: all 1s;
}
.current-name .letter.in-state {
  color: gold;
  font-weight: bold;
}
.current-name .letter.not-in-state + .letter.in-state {
  margin-left: 2em;
}
.current-name .letter.in-state + .letter.not-in-state {
  margin-left: 2em;
}
</style>
