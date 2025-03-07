<template>
  <div id="app">
    <StartScreen
      v-if="currentScreen === 'start'"
      @start-game="switchToGame"
      @show-how-to-play="showHowToPlay"
    />
    <GrammarQuizImage
      v-else-if="currentScreen === 'game'"
      @back-to-start="switchToStart"
    />
    <HowToPlayPopup
  v-if="isHowToPlayVisible"
  :isVisible="isHowToPlayVisible"
  @close="hideHowToPlay"
/>
  </div>
</template>

<script>
import StartScreen from './components/StartScreen.vue';
import GrammarQuizImage from './components/GrammarQuizImage.vue';
import HowToPlayPopup from './components/HowToPlayPopup.vue';

export default {
  components: {
    StartScreen,
    GrammarQuizImage,
    HowToPlayPopup
  },
  data() {
    return {
      currentScreen: 'start', // 'start' または 'game'
      isHowToPlayVisible: false
    };
  },
  methods: {
    switchToGame() {
      this.currentScreen = 'game';
    },
    switchToStart() {
      this.currentScreen = 'start';
    },
    showHowToPlay() {
    console.log('showHowToPlayが呼ばれました'); // デバッグ用
    this.isHowToPlayVisible = true; // 状態を更新
    console.log('isHowToPlayVisible:', this.isHowToPlayVisible); // 状態を確認
  },
    hideHowToPlay() {
      this.isHowToPlayVisible = false;
    }
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>