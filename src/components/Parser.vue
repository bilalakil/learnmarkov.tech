<template>
  <div id="parser">
    <transition v-if="currentWord" appear name="fade">
      <div :key="nameCursor" class="fade-absolute fade-absolute-top">
        <span
          v-for="(word, wi) in wordsInCurrentName"
          :ref="wi === currentWordIndex ? 'currentWord' : null"
          :class="[wi === currentWordIndex ? 'current-word' : 'stopped-word']"
          class="word"
        >
          <template v-if="wi === currentWordIndex">
            <span v-for="(letter, i) in currentWord"
              class="letter"
              :class="{ 'in-left-state': indexInState(i, 'left'),
                        'not-in-left-state': !indexInState(i, 'left'),
                        'in-right-state': indexInState(i, 'right'),
                        'not-in-right-state': !indexInState(i, 'right') }"
            >{{ letter }}</span>
            <span class="letter"></span>
          </template>
          <template v-else>
            {{ word }}
          </template>
        </span>
        <div id="possible-states">
          <transition appear name="fade">
            <div
              :key="currentState"
              class="fade-absolute fade-absolute-right"
            >
              <transition-group
                appear name="fade"
                @before-enter="moveFromRightState" @after-enter="clearMove"
              >
                <template v-if="typeof transitions[currentState] !== 'undefined'">
                  <span
                    class="state fade-move" :key="state"
                    v-for="state in transitions[currentState]"
                  >{{ state === '' ? '&lt;END&gt;' : state }}</span>
                </template>
              </transition-group>
              <span class="state placeholder" ref="possibleStatePlaceholder"></span>
            </div>
          </transition>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
let curLoop = 0

const stopWords = [
  '[^A-Z]',
  'PARTNERSHIP',
  'HOLDINGS',
  'PTY',
  'LTD',
  'LIMITED',
  'GROUP',
  'AUSTRALIA',
  'AUSTRALIAN',
  'AUSTRALASIA',
  'CORPORATION',
  'ASIA',
  'PACIFIC',
  'COMPANY',
  'INTERNATIONAL'
]
const stopWordsRe = RegExp(`( ?\\(?(${stopWords.join('|')})\\)?)+$`)

export default {
  name: 'parser',
  props: {
    data: Array,
    settings: Object
  },
  data () {
    return {
      nextStep: this.nextName,

      animated: true,

      starts: [],
      transitions: {},

      nameCursor: -1,

      wordsInCurrentName: null,
      wordStatus: null,
      currentWordIndex: -1,

      stateCursor: -1
    }
  },
  computed: {
    currentWord () {
      if (this.currentWordIndex !== -1) {
        return this.wordsInCurrentName[this.currentWordIndex]
      }
    },
    currentStoppedName () {
      if (this.nameCursor !== -1 && this.nameCursor < this.data.length) {
        let name = this.data[this.nameCursor]
        const match = name.match(stopWordsRe)

        if (match) {
          return match[0].trim()
        }
      }
    },
    currentState () {
      if (this.currentWord &&
          this.stateCursor !== -1 &&
          this.stateCursor + this.settings.stateWidthLeft <= this.currentWord.length
      ) {
        return this.currentWord.slice(this.stateCursor, this.stateCursor + this.settings.stateWidthLeft)
      }
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
        if (!self.settings.playing || !self.animated) {
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

    nextName () {
      const self = this

      while (true) {
        this.nameCursor += 1

        if (this.nameCursor >= this.data.length) {
          break
        }

        this.wordsInCurrentName = this.data[this.nameCursor].split(' ')
          .map(Function.prototype.call, String.prototype.trim)

        if (this.wordsInCurrentName.some(word => {
          return word.length > this.settings.stateWidthLeft && self.wordIsGood(word)
        })) {
          break
        }
      }

      this.stateCursor = -1

      if (this.nameCursor < this.data.length) {
        this.wordStatus = this.wordsInCurrentName.map(this.wordIsGood)
        this.currentWordIndex = this.wordStatus.indexOf(true)

        this.nextStep = this.nextState
      } else {
        this.wordsInCurrentName = null
        this.wordStatus = null
        this.currentWordIndex = -1

        this.nextStep = this.complete
      }

      this.next()
    },
    nextState () {
      this.stateCursor += 1

      if (this.stateCursor + this.settings.stateWidthLeft > this.currentWord.length) {
        this.nextStep = this.nextName
      } else {
        this.nextStep = this.registerPossibleState
      }

      this.next()
    },
    registerPossibleState () {
      let ts = this.transitions[this.currentState]
      if (typeof ts === 'undefined') {
        this.$set(this.transitions, this.currentState, [])
        ts = this.transitions[this.currentState]
      }

      const pos = this.stateCursor + this.settings.stateWidthLeft
      ts.push(this.currentWord.slice(pos, pos + this.settings.stateWidthRight))

      if (this.stateCursor === 0) {
        this.starts.push(this.currentState)
      }

      if (this.stateCursor !== this.currentWord.length - this.settings.stateWidthLeft) {
        this.nextStep = this.nextState
      } else {
        this.nextStep = this.nextName
      }

      this.next()
    },
    complete () {
      this.nextStep = null

      this.$emit('complete', {
        starts: this.starts,
        transitions: this.transitions
      })
    },

    wordIsGood (word) {
      return !word.match(stopWordsRe)
    },
    indexInState (i, lor) {
      const shift = this.stateCursor + (lor === 'left' ? 0 : this.settings.stateWidthLeft)

      return this.stateCursor !== -1 &&
        i >= shift &&
        i < shift + (
          lor === 'left'
            ? this.settings.stateWidthLeft
            : this.settings.stateWidthRight
        )
    },
    moveFromRightState (el) {
      const i = this.stateCursor + this.settings.stateWidthLeft
      const letter = this.$refs.currentWord[0].querySelectorAll('.letter')[i]

      const srcPos = letter.getBoundingClientRect()
      const tarPos = this.$refs.possibleStatePlaceholder.getBoundingClientRect()

      el.style.setProperty('--moveX', srcPos.left - tarPos.left)
      el.style.setProperty('--moveY', srcPos.bottom - tarPos.top)
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
      this.nextStep = this.nextName

      this.animated = true
      this.playing = true

      this.starts = []
      this.transitions = {}

      this.nameCursor = -1

      this.wordsInCurrentName = null
      this.wordStatus = null
      this.currentWordIndex = -1

      this.stateCursor = -1

      this.next()
    },
    stopAnimation () {
      this.animated = false

      if (this.nameCursor === -1) {
        this.nameCursor = 0
      }

      while (this.nextStep !== null) {
        this.nextStep()
      }
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
    },
    'settings.stateWidthLeft' () {
      this.restart()
    },
    'settings.stateWidthRight' () {
      this.restart()
    }
  }
}
</script>
