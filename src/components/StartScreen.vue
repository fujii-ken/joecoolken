<template>
  <div class="start-screen">
    <div class="background"></div>
    <div class="content">
      <h1>桜の並び替えゲーム</h1>
      <button @click="startGame">スタート</button>
      <button @click="showHowToPlay">遊び方</button>
    </div>
  </div>
</template>

<script>
export default {
  methods: {
    startGame() {
      this.$emit('start-game');

    },
    showHowToPlay() {
      this.$emit('show-how-to-play');
      console.log('遊び方ボタンがクリックされました'); 
    }
  }
};
</script>

<style scoped>
.start-screen {
  position: relative;
  width: 100%;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f0f0f0;
}

.background {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-image: url('~@/assets/sakura-background.jpg');
  background-size: cover;
  opacity: 0.6;
  z-index: 0;
}

.content {
  position: relative;
  z-index: 1;
  text-align: center;
}

h1 {
  font-size: 3rem;
  color: #333;
  margin-bottom: 2rem;
}

button {
  font-size: 1.5rem;
  padding: 1rem 2rem;
  margin: 0.5rem;
  border: none;
  border-radius: 5px;
  background-color: #ff6f61;
  color: white;
  cursor: pointer;
  transition: background-color 0.3s;
}

button:hover {
  background-color: #ff3b2f;
}
</style>