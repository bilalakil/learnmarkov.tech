<template>
  <div id="parser">
    <transition v-if="currentName" appear name="fade">
      <div :key="currentName" class="fade-absolute fade-absolute-top">
        <div id="current-name" ref="currentName">
          <span v-for="(letter, i) in currentName"
            class="letter"
            :class="{ 'in-left-state': indexInState(i, 'left'),
                      'not-in-left-state': !indexInState(i, 'left'),
                      'in-right-state': indexInState(i, 'right'),
                      'not-in-right-state': !indexInState(i, 'right') }"
          >{{ letter }}</span>
          <span class="letter"></span>
        </div>
        <transition appear name="fade">
          <div id="stopped-name">
            {{ currentStoppedName }}
          </div>
        </transition>

        <transition appear name="fade">
          <div
            id="possible-states"
            :key="currentState"
            class="fade-absolute fade-absolute-right"
          >
            <transition-group
              appear name="fade"
              @before-enter="moveFromRightState" @after-enter="clearMove"
            >
              <template v-if="typeof stateTransitions[currentState] !== 'undefined'">
                <span
                  class="state fade-move" :key="state"
                  v-for="state in stateTransitions[currentState]"
                >{{ state === '' ? '&lt;END&gt;' : state }}</span>
              </template>
            </transition-group>
            <span class="state placeholder" ref="possibleStatePlaceholder"></span>
          </div>
        </transition>
      </div>
    </transition>
  </div>
</template>

<script>
let curLoop = 0

const stopWords = [
  'PARTNERSHIP',
  'HOLDINGS',
  'PTY',
  'LTD',
  'LIMITED',
  'GROUP',
  'AUSTRALIA',
  'CORPORATION',
  'AUSTRALASIA',
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

      stateTransitions: {},

      nameCursor: -1,
      stateCursor: -1
    }
  },
  computed: {
    currentName () {
      if (this.nameCursor !== -1 && this.nameCursor < this.data.length) {
        let name = this.data[this.nameCursor]
        const match = name.match(stopWordsRe)

        if (match) {
          name = name.slice(0, name.length - match[0].length)
        }

        return name.trim().replace(/ /g, '_')
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
      if (this.currentName &&
          this.stateCursor !== -1 &&
          this.stateCursor + this.settings.stateWidthLeft <= this.currentName.length
      ) {
        return this.currentName.slice(this.stateCursor, this.stateCursor + this.settings.stateWidthLeft)
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
      while (true) {
        this.nameCursor += 1

        if (this.nameCursor >= this.data.length ||
            this.currentName.length >= this.settings.stateWidthLeft) {
          break
        }
      }
      this.stateCursor = -1

      if (this.nameCursor < this.data.length) {
        this.nextStep = this.nextState
      } else {
        this.nextStep = this.complete
      }

      this.next()
    },
    nextState () {
      this.stateCursor += 1

      if (this.stateCursor + this.settings.stateWidthLeft > this.currentName.length) {
        this.nextStep = this.nextName
      } else {
        this.nextStep = this.registerPossibleState
      }

      this.next()
    },
    registerPossibleState () {
      let ts = this.stateTransitions[this.currentState]
      if (typeof ts === 'undefined') {
        this.$set(this.stateTransitions, this.currentState, [])
        ts = this.stateTransitions[this.currentState]
      }

      const pos = this.stateCursor + this.settings.stateWidthLeft
      ts.push(this.currentName.slice(pos, pos + this.settings.stateWidthRight))

      if (this.stateCursor !== this.currentName.length - this.settings.stateWidthLeft) {
        this.nextStep = this.nextState
      } else {
        this.nextStep = this.nextName
      }

      this.next()
    },
    complete () {
      this.nextStep = null

      this.$emit('complete', this.stateTransitions)
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
      const letter = this.$refs.currentName.querySelectorAll('.letter')[i]

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

      this.stateTransitions = {}

      this.nameCursor = -1
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
