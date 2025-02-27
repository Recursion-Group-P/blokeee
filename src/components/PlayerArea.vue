<template>
  <div
    class="rounded-borders shadow-1 q-my-sm q-mx-auto q-pa-md player-area relative-position"
    :class="{
      'no-pointer-events': players[playerId].outOfGame === true,
      disabled: players[playerId].outOfGame === true,
    }"
    :style="`background-color: ${playerColor}; width: ${numberOfPlayers === 2 ? '100%' : '80%'}`"
  >
    <div class="row justify-between">
      <h5 class="q-ma-none">
        <strong>{{ this.players[this.playerId].name }}</strong>
      </h5>
      <h6 class="q-ma-none player-score">
        {{ 'Score: ' + this.players[this.playerId].score }}
      </h6>
    </div>
    <div class="row justify-between">
      <div class="q-mb-md row items-center" :class="{ 'text-red-8': time <= 10 && time >= 1 }">
        <q-icon name="timer" size="1.5rem"></q-icon>
        <h6 class="q-ma-none">{{ formatTime }}</h6>
      </div>
      <div>
        <q-btn
          v-if="!players[playerId].isAI"
          filled
          rounded
          color="white"
          text-color="blue-grey"
          size="sm"
          padding="sm md"
          label="Pass"
          @click="toggleConfirmPassArea"
          :disabled="confirmPassArea || currentPlayerId !== playerId"
        />
      </div>
    </div>
    <piece-selector
      @select-piece="selectPiece"
      v-if="currPlayerSelectedPieceId === -1 && confirmPassArea === false"
      :player-id="playerId"
    />
    <piece-alter
      @cancel-piece="selectPiece"
      v-if="currPlayerSelectedPieceId !== -1 && confirmPassArea === false"
      :player-id="playerId"
    />
    <confirm-pass
      v-if="confirmPassArea === true"
      @hideConfirmPassArea="toggleConfirmPassArea"
      @stopTimer="stopTimer"
      @passPlayerTurn="passPlayerTurn"
      :player-id="playerId"
    />
    <div
      v-if="players[playerId].outOfGame === true"
      class="absolute-center bg-white rounded-borders text-center shadow-1 q-pa-md"
    >
      <div class="q-mb-sm">Please wait until the game ends...</div>
      <q-icon name="coffee" size="1.3rem"></q-icon>
    </div>
  </div>
</template>

<script>
// Components
import PieceAlter from './PieceAlter.vue';
import PieceSelector from './PieceSelector.vue';
import ConfirmPass from './ConfirmPass.vue';
// Constants
import { PLAYER_COLORS } from 'src/constants';
// Vuex
import { mapGetters, mapActions } from 'vuex';
// Vue
import Vue from 'vue';

export default Vue.extend({
  props: ['playerId'],
  components: {
    'piece-alter': PieceAlter,
    'piece-selector': PieceSelector,
    'confirm-pass': ConfirmPass,
  },
  data() {
    return {
      // For timer
      time: 0,
      timerObj: null,
      selectedPieceId: -1,
      // For Confirm Pass
      confirmPassArea: false,
      isWatching: [true, true],
      outOfGameWatcher: null,
    };
  },
  computed: {
    ...mapGetters('game', [
      'players',
      'numberOfPlayers',
      'timeForEachPlayer',
      'currentPlayerId',
      'gameIsOver',
    ]),

    thisPlayerOutOfGame() {
      return this.players[this.playerId].outOfGame;
    },

    currPlayerSelectedPieceId() {
      return this.players[this.playerId].selectedPieceId;
    },

    formatTime() {
      let min = Math.floor(this.time / 60);
      let sec = ('00' + (this.time % 60)).slice(-2);
      let formatTime = `${min}:${sec}`;
      return formatTime;
    },

    playerColor() {
      return this.currentPlayerId === this.playerId
        ? this.players[this.playerId].color
        : this.players[this.playerId].color + '66'; //TODO: indicator for current player
    },
  },
  methods: {
    ...mapActions('game', ['updatePlayerOutOfGame', 'recordRemainingTime']),
    // for timer
    countDown() {
      this.time--;
    },
    startTimer() {
      let self = this;
      this.timerObj = setInterval(function () {
        self.countDown();
      }, 1000);
    },
    stopTimer() {
      // console.log()
      clearInterval(this.timerObj);
      this.recordRemainingTime({
        currentPlayerId: this.playerId,
        remainingTime: this.time,
      });
    },
    selectPiece(e) {
      this.selectedPieceId = e;
    },
    toggleConfirmPassArea() {
      this.confirmPassArea = !this.confirmPassArea;
    },
    passPlayerTurn() {
      this.$emit('passPlayerTurn');
    },
  },
  mounted() {
    this.time = this.timeForEachPlayer;
  },
  watch: {
    // Timerが0になったらstore-gameのmutationでplayerのoutOfGameをtrueに更新
    time: {
      handler(time) {
        if (time <= 0) {
          this.stopTimer();
          this.updatePlayerOutOfGame({ currentPlayerId: this.currentPlayerId });
          this.$emit('passPlayerTurn');
        }
      },
    },
    currentPlayerId: {
      handler(currentPlayerId) {
        if (currentPlayerId === this.playerId) this.startTimer();
        else this.stopTimer();
      },
      immediate: true,
    },
  },
});
</script>

<style scoped>
.player-area {
  touch-action: manipulation;
}

@media screen and (max-width: 768px) {
  .player-area {
    width: 100% !important;
  }
}

.player-score {
  font-size: 16px;
}
</style>
