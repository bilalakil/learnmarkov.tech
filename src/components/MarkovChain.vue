<template>
  <div id="markov-chain">
    <template v-if="controllerSettings">
      <parser :settings="controllerSettings" :data="data" ref="activeStage"></parser>
    </template>

    <controller
      @settings="controllerSettings = $event"
      @restart="forwardInstr('restart')"
      @nextStep="forwardInstr('nextStep')"
      @stopAnimation="forwardInstr('stopAnimation')"
    ></controller>
  </div>
</template>

<script>
import Parser from './Parser'
import Controller from './Controller'

export default {
  name: 'markov-chain',
  props: {
    data: Array
  },
  components: {
    Parser,
    Controller
  },
  data () {
    return {
      controllerSettings: null
    }
  },
  methods: {
    forwardInstr (instr) {
      this.$refs.activeStage.handle(instr)
    }
  }
}
</script>

<style>
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
