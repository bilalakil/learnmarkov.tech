<template>
  <div id="generator">
    <div v-if="currentWord" id="current-name">
      <transition appear>
        <div
          :class="{ 'finished': complete }"
        >
          <span
            v-for="(word, wi) in wordsInCurrentName"
            class="word"
            :class="{ 'stopped-word': hasBusinessWord && wi === wordsInCurrentName.length - 1 }"
          >
            <transition-group appear>
              <span v-for="(letter, i) in word"
                :key="i" :data-key="i"
                class="letter"
                :class="{ 'in-left-state': indexInState(i),
                          'not-in-left-state': !indexInState(i) }"
              >{{ letter }}</span>
            </transition-group>
          </span>
        </div>
      </transition>

      <div
        v-if="currentState" id="possible-states"
        class="fade-absolute-container"
      >
        <transition appear>
          <div
            :key="currentState"
            class="fade-absolute fade-absolute-right"
          >
            <transition-group
              appear
            >
              <span
                class="state" :key="currentState + i"
                :class="{ selected: i === nextState }"
                v-for="(state, i) in states.transitions[currentState]"
                :ref="i === nextState ? 'selectedState' : null"
              >{{ state }}</span>
            </transition-group>
          </div>
        </transition>
      </div>
    </div>
  </div>
</template>

<script>
let curLoop = 0

const businessWords = [
  'LIMITED',
  'PTY LTD',
  'CORPORATION',
  'HOLDINGS',
  'INTERNATIONAL',
  'GROUP'
]

const randNumWords = () => Math.floor(Math.random() * 3) + 1
const randWordLength = () => Math.floor(Math.random() * 7) + 4

const getDefaultData = function () {
  return {
    nextStep: this.startWord,

    targetNumWords: null,
    targetWordLength: null,

    wordsInCurrentName: [],
    currentStateIndex: -1,
    stateLengthLeft: null,
    nextState: null,
    complete: false,
    hasBusinessWord: false
  }
}

export default {
  name: 'generator',
  props: {
    data: Array,
    states: Object,
    settings: Object
  },
  data: getDefaultData,
  computed: {
    currentWord () {
      if (this.wordsInCurrentName.length !== 0) {
        return this.wordsInCurrentName[this.wordsInCurrentName.length - 1]
      }
    },
    currentState () {
      if (this.currentStateIndex === -1 || !this.currentWord || !this.stateLengthLeft) {
        return
      }

      return this.currentWord.slice(this.currentStateIndex, this.currentStateIndex + this.stateLengthLeft)
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

    startWord () {
      if (this.wordsInCurrentName.length === 0) {
        this.targetNumWords = randNumWords()
      } else if (this.wordsInCurrentName.length === this.targetNumWords) {
        this.finish()
        return
      }

      const states = this.settings.strictStarts
        ? this.states.starts
        : Object.keys(this.states.transitions)

      this.targetWordLength = randWordLength()

      this.wordsInCurrentName.push(states[Math.floor(Math.random() * states.length)])
      this.currentStateIndex = 0
      this.stateLengthLeft = this.currentWord.length

      this.nextStep = this.selectNextState
      this.next()
    },
    selectNextState () {
      const connectingStates = this.states.transitions[this.currentState]

      if (!connectingStates) {
        this.finish()
        return
      }

      this.nextState = Math.floor(Math.random() * connectingStates.length)

      this.nextStep = this.useNextState
      this.next()
    },
    useNextState () {
      const next = this.states.transitions[this.currentState][this.nextState]

      this.nextState = null

      this.wordsInCurrentName.splice(this.wordsInCurrentName.length - 1, 1, this.currentWord + next)

      if (this.currentWord.length >= this.targetWordLength) {
        this.nextStep = this.prepareForNextWord
      } else {
        this.nextStep = this.advanceCurrentState
      }

      this.next()
    },
    advanceCurrentState () {
      this.currentStateIndex = this.currentWord.length - this.stateLengthLeft

      this.nextStep = this.selectNextState
      this.next()
    },
    prepareForNextWord () {
      this.currentStateIndex = -1

      this.nextStep = this.startWord
      this.next()
    },
    finish () {
      if (Math.random() >= 0.5) {
        this.wordsInCurrentName.push(businessWords[Math.floor(Math.random() * businessWords.length)])
        this.hasBusinessWord = true
      }

      this.currentStateIndex = -1
      this.complete = true

      this.nextStep = this.restart
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
      const defaults = getDefaultData.call(this)
      for (let k in defaults) {
        this.$set(this, k, defaults[k])
      }

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
