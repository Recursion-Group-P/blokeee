<template>
  <q-page padding>
    <h2 class="text-center">Game Settings</h2>
    <div class="text-h6 text-center">プレイ人数・持ち時間・開始位置を選んでください。</div>
    <div class="row justify-center q-mt-lg">
      <div class="col-10 col-md-6 col-lg-5 text-center">
        <q-card class="my-card">
          <q-card-section>
            <q-select
              v-model="numberOfPlayers"
              :options="numberOfPlayersOptions"
              label="Number of players"
            />
            <q-select
              v-model="numberOfCPU"
              :options="numberOfCPUOptions[numberOfPlayers]"
              label="Number of CPU"
            />
            <div class="row">
              <div class="q-mr-md col-5 col-sm-2" v-for="n of numberOfCPU" :key="n">
                <q-select
                  v-model="CPUStrength[n - 1]"
                  :options="CPUStrengthOptions"
                  :label="`CPU ` + n"
                />
              </div>
            </div>
            <q-select
              v-model="timeForEachPlayer"
              :options="timeForEachPlayerOptions"
              label="Time for each player"
            />
            <q-select
              v-model="startPosition"
              :options="startPositionOptions"
              label="Start position"
            />
            <q-btn
              class="q-mt-lg full-width"
              @click="gameStart"
              to="/room"
              unelevated
              size="lg"
              color="black"
              label="Play"
            />
          </q-card-section>
        </q-card>
      </div>
    </div>
  </q-page>
</template>

<script>
import { Config } from '../config/index';
import { mapActions } from 'vuex';
import Vue from 'vue';

export default Vue.extend({
  name: 'SettingsPage',
  data() {
    return {
      numberOfPlayers: Config.numberOfPlayers,
      numberOfPlayersOptions: Config.numberOfPlayersOptions,
      numberOfCPU: Config.numberOfCPU,
      numberOfCPUOptions: Config.numberOfCPUOptions,
      CPUStrength: Config.CPUStrength,
      CPUStrengthOptions: Config.CPUStrengthOptions,
      timeForEachPlayer: Config.timeForEachPlayer,
      timeForEachPlayerOptions: Config.timeForEachPlayerOptions,
      startPosition: Config.startPosition,
      startPositionOptions: Config.startPositionOptions,
    };
  },
  methods: {
    ...mapActions('game', ['setGameSettings', 'formatState']),
    gameStart() {
      console.log({
        numberOfPlayers: this.numberOfPlayers,
        timeForEachPlayer: this.timeForEachPlayer['value'],
        startPosition: this.startPosition,
        numberOfCPU: this.numberOfCPU,
        CPUStrength: this.CPUStrength,
      });
      let totalCells = (this.numberOfPlayers == 4) ? 20 : 14;
      this.setGameSettings({
        numberOfPlayers: this.numberOfPlayers,
        timeForEachPlayer: this.timeForEachPlayer['value'],
        startPosition: this.startPosition,
        numberOfCPU: this.numberOfCPU,
        CPUStrength: this.CPUStrength,
        totalCells: totalCells
      });
    },
  },
  mounted() {
    // 前のゲームのデータを削除
    this.formatState();
  },
});
</script>

<style></style>
