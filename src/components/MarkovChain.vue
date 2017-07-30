<template>
  <div id="markov-chain" :style="{ '--delay': controllerSettings ? controllerSettings.delay : 1000 }">
    <div className="padder"></div>
    <transition v-if="controllerSettings" name="fade">
      <parser ref="activeStage" v-if="!possibleStates"
        :settings="controllerSettings"
        :data="data"
        @complete="possibleStates = $event"
      ></parser>
      <generator ref="activeStage" v-else
        :settings="controllerSettings"
        :data="data" :states="possibleStates"
      ></generator>
    </transition>
    <controller
      :mode="possibleStates ? 'generator' : 'parser'"
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
      possibleStates: null
    }
  },
  methods: {
    forwardInstr (instr) {
      this.$refs.activeStage.handle(instr)
    },
    clearData () {
      this.possibleStates = null
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

#current-name .letter,
#possible-states .state,
.fade-enter-active, .fade-leave-active {
  transition: all calc(var(--delay) * 1ms);
}

#current-name { display: inline-block; }
#current-name .letter { display: inline-block; }
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
#possible-states .state.selected {
  color: gold;
  font-weight: bold;
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
