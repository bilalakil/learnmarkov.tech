<template>
  <div id="parser" class="fade-absolute-container">
    <transition v-if="currentWord" appear>
      <div :key="nameCursor" class="fade-absolute fade-absolute-top">
        <span ref="word"
          v-for="(word, wi) in wordsInCurrentName"
          :class="{ 'current-word': wi === currentWordIndex,
                    'stopped-word': !wordStatus[wi] }"
          class="word"
        >
          <template v-if="wordStatus[wi]">
            <span v-for="(letter, i) in word"
              class="letter"
              :class="{ 'in-left-state': indexInState(wi, i, 'left'),
                        'not-in-left-state': !indexInState(wi, i, 'left'),
                        'in-right-state': indexInState(wi, i, 'right'),
                        'not-in-right-state': !indexInState(wi, i, 'right') }"
            >{{ letter }}</span>
            <span class="letter"></span>
          </template>
          <template v-else>
            {{ word }}
          </template>
        </span>

        <div
          id="possible-states" class="fade-absolute-container"
          v-if="stateCursor + settings.stateWidthLeft !== currentWord.length"
        >
          <transition appear>
            <div
              :key="currentState"
              class="fade-absolute fade-absolute-right"
            >
              <transition-group
                appear
                @before-enter="moveFromRightState" @after-enter="clearMove"
              >
                <template v-if="typeof transitions[currentState] !== 'undefined'">
                  <span
                    class="state fade-move" :key="state"
                    v-for="(state, i) in transitions[currentState]"
                    key="i"
                    :data-key="i"
                  >{{ state }}</span>
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
  '[A-Z]*[^A-Z]+[A-Z]*',
  'PARTNERSHIP',
  'HOLDINGS?',
  'PTY',
  'LTD',
  'LIMITED',
  'GROUP',
  'AUSTRALIAN?',
  'AUSTRALASIA',
  'CORPORATION',
  'ASIA',
  'PACIFIC',
  'COMPANY',
  'INTERNATIONAL'
]
const stopWordsRe = RegExp(`^\\(?(${stopWords.join('|')})\\)?$`)

const getDefaultData = function () {
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
}

export default {
  name: 'parser',
  props: {
    data: Array,
    settings: Object
  },
  data: getDefaultData,
  computed: {
    currentWord () {
      if (this.currentWordIndex !== -1) {
        return this.wordsInCurrentName[this.currentWordIndex]
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

      const endPos = this.stateCursor + this.settings.stateWidthLeft

      if (endPos >= this.currentWord.length) {
        const nextIndex = this.wordStatus.slice(this.currentWordIndex + 1)
          .indexOf(true)

        if (nextIndex !== -1) {
          if (endPos === this.currentWord.length) {
            this.nextStep = this.nextState
          } else {
            this.currentWordIndex += nextIndex + 1
            this.stateCursor = 0
            this.nextStep = this.registerPossibleState
            this.next(2)
            return
          }
        } else {
          this.nextStep = this.nextName
        }
      } else {
        this.nextStep = this.registerPossibleState
      }

      this.next()
    },
    registerPossibleState () {
      if (!this.currentState) {
        this.nextStep = this.nextName
        this.next()
        return
      }

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

      this.nextStep = this.nextState
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
      return word.length > this.settings.stateWidthLeft && !word.match(stopWordsRe)
    },
    indexInState (wi, i, lor) {
      const shift = this.stateCursor + (lor === 'left' ? 0 : this.settings.stateWidthLeft)

      return this.stateCursor !== -1 &&
        wi === this.currentWordIndex &&
        i >= shift &&
        i < shift + (
          lor === 'left'
            ? this.settings.stateWidthLeft
            : this.settings.stateWidthRight
        )
    },
    moveFromRightState (el) {
      const placeholder = this.$refs.possibleStatePlaceholder
      if (!placeholder) {
        return
      }

      const i = this.stateCursor + this.settings.stateWidthLeft
      const letter = this.$refs.word[this.currentWordIndex].querySelectorAll('.letter')[i]

      const srcPos = letter.getBoundingClientRect()
      const tarPos = placeholder.getBoundingClientRect()

      el.style.setProperty('--moveX', srcPos.left - tarPos.left + 40)
      el.style.setProperty('--moveY', srcPos.bottom - tarPos.top + 20)
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
