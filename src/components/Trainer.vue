<template>
  <div :style="{ '--delay': delay }">
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
          <div id="possible-states" :key="currentState" class="fade-absolute fade-absolute-right">
            <transition-group appear name="fade" @before-enter="moveFromRightState" @after-enter="clearMove">
              <template v-if="typeof stateTransitions[currentState] !== 'undefined'">
                <span class="state fade-move" :key="state"
                  v-for="state in stateTransitions[currentState]"
                >{{ state === '' ? '&lt;END&gt;' : state }}</span>
              </template>
            </transition-group>
            <span class="state placeholder" ref="possibleStatePlaceholder"></span>
          </div>
        </transition>
      </div>
    </transition>
    <div id="control">
      <label>
        Speed:
        <input type="range" v-model.number="speed" min="1" v-bind:max="100"/>
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
const animSpeed = 5000

const stopWords = [
  'HOLDINGS',
  'PTY',
  'LTD',
  'LIMITED',
  'GROUP',
  'AUSTRALIA',
  'CORPORATION',
  'AUSTRALASIA',
  'ASIA PACIFIC',
  'COMPANY',
  'INTERNATIONAL'
]
const stopWordsRe = RegExp(`( ?\\(?(${stopWords.join('|')})\\)?)+$`)

export default {
  name: 'trainer',
  props: {
    data: Array
  },
  data () {
    return {
      speed: 5,
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
          this.stateCursor + this.stateWidthLeft <= this.currentName.length
      ) {
        return this.currentName.slice(this.stateCursor, this.stateCursor + this.stateWidthLeft)
      }
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
      while (true) {
        this.nameCursor += 1

        if (this.nameCursor >= this.data.length ||
            this.currentName.length >= this.stateWidthLeft
        ) {
          break
        }
      }
      this.stateCursor = -1

      if (this.nameCursor < this.data.length) {
        this.nextStep = this.nextState
        this.next()
      }
    },
    nextState () {
      this.stateCursor += 1

      this.nextStep = this.registerPossibleState
      this.next()
    },
    registerPossibleState () {
      let ts = this.stateTransitions[this.currentState]
      if (typeof ts === 'undefined') {
        this.$set(this.stateTransitions, this.currentState, [])
        ts = this.stateTransitions[this.currentState]
      }

      const pos = this.stateCursor + this.stateWidthLeft
      ts.push(this.currentName.slice(pos, pos + this.stateWidthRight))

      if (this.stateCursor !== this.currentName.length - this.stateWidthLeft) {
        this.nextStep = this.nextState
      } else {
        this.nextStep = this.nextName
      }

      this.next()
    },

    indexInState (i, lor) {
      const shift = this.stateCursor + (lor === 'left' ? 0 : this.stateWidthRight)

      return this.stateCursor !== -1 &&
        i >= shift &&
        i < shift + this.stateWidthLeft
    },
    moveFromRightState (el) {
      const i = this.stateCursor + this.stateWidthLeft
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

    restart () {
      this.nextStep = this.nextName

      this.stateTransitions = {}

      this.nameCursor = -1
      this.stateCursor = -1

      this.next(1.5)
    }
  },
  mounted () {
    this.next(1.5)
  }
}
</script>

<style scoped>
#current-name { display: inline-block; }
#current-name .letter {
  display: inline-block;

  transition: all calc(var(--delay) * 1ms);
}
#current-name .letter.in-left-state {
  color: gold;
  font-weight: bold;
}
#current-name .letter.in-right-state {
  color: blue;
  font-weight: bold;
}
#current-name .letter.not-in-left-state + .letter.in-left-state { margin-left: 2em; }
#current-name .letter.in-left-state + .letter.not-in-left-state { margin-left: 2em; }
#current-name .letter.in-right-state + .letter.not-in-right-state { margin-left: 1em; }

#stopped-name {
  display: inline-block;
  margin-left: 4em;

  opacity: 0.5;
}

#possible-states .state {
  display: inline-block;
  margin-right: 2em;
}

.fade-enter-active, .fade-leave-active { transition: all calc(var(--delay) * 1ms); }
.fade-enter, .fade-leave-to { opacity: 0; }
.fade-absolute.fade-leave-active { position: absolute; }
.fade-absolute-top.fade-enter { transform: translateX(-2em); }
.fade-absolute-top.fade-leave-to { transform: translateX(2em); }
.fade-absolute-right.fade-enter { transform: translateX(-4em); }
.fade-absolute-right.fade-leave-to { transform: translateX(4em); }
.fade-move { transform: translate(calc(var(--moveX) * 1px), calc(var(--moveY) * 1px)); }
.fade-move.fade-enter-to { transform: translate(0, 0); }
</style>
