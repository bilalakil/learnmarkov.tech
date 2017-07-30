<template>
  <div id="markov-chain" :style="{ '--delay': controllerSettings ? controllerSettings.delay : 1000 }">
    <div className="padder"></div>
    <transition v-if="controllerSettings" name="fade">
      <parser ref="activeStage" v-if="!states"
        :settings="controllerSettings"
        :data="data"
        @complete="states = $event"
      ></parser>
      <generator ref="activeStage" v-else
        :settings="controllerSettings"
        :data="data" :states="states"
      ></generator>
    </transition>
    <controller
      :mode="states ? 'generator' : 'parser'"
      @settings="controllerSettings = $event"
      @restart="forwardInstr('restart')"
      @nextStep="forwardInstr('nextStep')"
      @stopAnimation="forwardInstr('stopAnimation')"
      @clearData="clearData"
    ></controller>
  </div>
</template>

<script>
import Parser from './Parser'
import Generator from './Generator'
import Controller from './Controller'

export default {
  name: 'markov-chain',
  props: {
    data: Array
  },
  components: {
    Parser,
    Generator,
    Controller
  },
  data () {
    return {
      controllerSettings: null,
      states: null
    }
  },
  methods: {
    forwardInstr (instr) {
      this.$refs.activeStage.handle(instr)
    },
    clearData () {
      this.states = null
    }
  }
}
</script>

<style>
#markov-chain {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  height: 100%;
}

#current-name, #stopped-name, #possible-states, .word {
  font-family: 'Droid Sans Mono', monospace;
  font-size: 20px;
}

#possible-states {
  font-size: 20px;
  margin-top: 20px;
  max-height: 50vh;
  position: relative;
}

#possible-states.generator {
  font-size: 16px;
  overflow: hidden;
}

.current-word .letter,
#possible-states .state,
.fade-enter-active, .fade-leave-active {
  transition: all calc(var(--delay) * 1ms);
}

.word {
  display: inline-block;
  margin-right: 2em;
}
.current-word .letter { display: inline-block; }
.current-word .letter.in-left-state {
  color: #edf177;
  font-weight: bold;
}
.current-word .letter.in-right-state {
  color: #90c9f5;
  font-weight: bold;
}
.current-word .letter.not-in-left-state + .letter.in-left-state { margin-left: 1em; }
.current-word .letter.in-left-state + .letter.not-in-left-state { margin-left: 1em; }
.current-word .letter.in-right-state + .letter.not-in-right-state { margin-left: 0.5em; }
.stopped-word { opacity: 0.5; }

#possible-states .state {
  display: inline-block;
  margin-right: 2em;
}
#possible-states .state.selected {
  color: #fd8383;
  font-weight: bold;
  font-size: 20px;
}

.fade-enter, .fade-leave-to { opacity: 0; }
.fade-absolute.fade-leave-active { position: absolute; }
.fade-absolute-top.fade-enter { transform: translateX(-2em); }
.fade-absolute-top.fade-leave-to { transform: translateX(2em); }
.fade-absolute-right.fade-enter { transform: translateX(-4em); }
.fade-absolute-right.fade-leave-to { transform: translateX(4em); }
.fade-move { transform: translate(calc(var(--moveX) * 1px), calc(var(--moveY) * 1px)); }
.fade-move.fade-enter-to { transform: translate(0, 0); }
</style>
