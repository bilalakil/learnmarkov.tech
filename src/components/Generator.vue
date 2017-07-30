<template>
  <div id="generator" class="fade-absolute-container">
    <transition v-if="currentName" appear name="fade">
      <div class="fade-absolute fade-absolute-top">
        <div id="current-name">
          <transition-group name="fade"
            @before-enter="moveToRightState" @after-enter="clearMove"
          >
            <span v-for="(letter, i) in currentName"
              :key="i" :data-key="i"
              class="letter fade-move"
              :class="{ 'in-left-state': indexInState(i),
                        'not-in-left-state': !indexInState(i) }"
            >{{ letter }}</span>
            <span key="placeholder"
              v-if="!complete"
              class="letter not-in-left-state in-right-state"
              ref="nextStatePlaceholder"
            >?</span>
          </transition-group>
        </div>
        <div id="possible-states" class="fade-absolute-container">
          <transition appear name="fade">
            <div :key="currentState" class="fade-absolute fade-absolute-right">
              <span
              class="state" :key="state"
              :class="{ selected: i === nextState }"
              v-for="(state, i) in states.transitions[currentState]"
              :ref="i === nextState ? 'selectedState' : null"
            >{{ state === '' ? '&lt;END&gt;' : state }}</span>
            </div>
          </transition>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
let curLoop = 0

// const numWords = () => { Math.floor(Math.random() * 3) + 1 }
// const wordLength = () => { Math.floor(Math.random() * 7) + 4 }

export default {
  name: 'generator',
  props: {
    data: Array,
    states: Object,
    settings: Object
  },
  data () {
    return {
      nextStep: this.startName,

      currentName: null,
      currentStateIndex: -1,
      stateLengthLeft: null,
      nextState: null,
      complete: false
    }
  },
  computed: {
    currentState () {
      if (this.currentStateIndex === -1 || !this.currentName || !this.stateLengthLeft) {
        return
      }

      return this.currentName.slice(this.currentStateIndex, this.currentStateIndex + this.stateLengthLeft)
    }
  },
  methods: {
    next (delayMult) {
      const self = this
      const thisLoop = ++curLoop
      const delayUntil = performance.now() + this.settings.delay * (
        typeof delayMult === 'undefined'
          ? 1
          : delayMult
      )

      const loop = (now) => {
        if (!self.settings.playing) {
          return
        }

        if (thisLoop !== curLoop) {
          return
        }

        if (now >= delayUntil) {
          self.nextStep()
        } else {
          requestAnimationFrame(loop)
        }
      }

      requestAnimationFrame(loop)
    },

    startName () {
      const states = this.settings.strictStarts
        ? this.states.starts
        : Object.keys(this.states.transitions)

      this.currentName = states[Math.floor(Math.random() * states.length)]
      this.currentStateIndex = 0
      this.stateLengthLeft = this.currentName.length

      this.nextStep = this.selectNextState
      this.next()
    },
    selectNextState () {
      const connectingStates = this.states.transitions[this.currentState]

      this.nextState = Math.floor(Math.random() * connectingStates.length)

      this.nextStep = this.useNextState
      this.next()
    },
    useNextState () {
      const next = this.states.transitions[this.currentState][this.nextState]

      this.nextState = null

      if (next !== '') {
        this.currentName += next

        this.nextStep = this.advanceCurrentState
        this.next()
      } else {
        this.currentStateIndex = -1
        this.complete = true

        this.nextStep = this.restart
      }
    },
    advanceCurrentState () {
      this.currentStateIndex = this.currentName.length - this.stateLengthLeft

      this.nextStep = this.selectNextState
      this.next()
    },

    indexInState (i) {
      return this.currentStateIndex !== -1 &&
        i >= this.currentStateIndex &&
        i < this.currentStateIndex + this.stateLengthLeft
    },
    moveToRightState (el) {
      const key = el.getAttribute('data-key')

      if (!key || key < this.currentStateIndex + this.stateLengthLeft) {
        return
      }

      const srcPos = this.$refs.selectedState[0].getBoundingClientRect()
      const tarPos = this.$refs.nextStatePlaceholder.getBoundingClientRect()

      el.style.setProperty('--moveX', srcPos.left - tarPos.left)
      el.style.setProperty('--moveY', srcPos.top - tarPos.top)
    },
    clearMove (el) {
      el.style.removeProperty('--moveX')
      el.style.removeProperty('--moveY')
    },

    handle (instr) {
      if (typeof this[instr] === 'function') {
        this[instr]()
      }
    },
    restart () {
      this.nextStep = this.startName

      this.currentName = null
      this.currentStateIndex = -1
      this.stateLengthLeft = null
      this.nextState = null
      this.complete = false

      this.next()
    }
  },
  mounted () {
    this.next()
  },
  watch: {
    'settings.playing' (val) {
      if (val) {
        this.next()
      }
    }
  }
}
</script>
