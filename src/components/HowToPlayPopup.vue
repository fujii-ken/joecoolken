<template>
  <div class="how-to-play-popup">
    <div class="overlay" @click="close"></div>
    <div class="popup-content">
      <button class="close-btn" @click="close">×</button>
      <div class="svg-container">
        <img :src="currentSvg" alt="How to Play" />
      </div>
      <div class="controls">
        <button @click="prevPage" :disabled="currentPage === 0">←</button>
        <button @click="nextPage" :disabled="currentPage === svgs.length - 1">→</button>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      svgs: [
        require('@/assets/vue.svg'),
        require('@/assets/vue.svg'),
        require('@/assets/vue.svg'),
        require('@/assets/vue.svg')
      ],
      currentPage: 0
    };
  },
  computed: {
    currentSvg() {
      return this.svgs[this.currentPage];
    }
  },
  methods: {
    close() {
      this.$emit('close'); // ポップアップを閉じる
    },
    prevPage() {
      if (this.currentPage > 0) {
        this.currentPage--;
      }
    },
    nextPage() {
      if (this.currentPage < this.svgs.length - 1) {
        this.currentPage++;
      }
    }
  },
  mounted() {
    window.addEventListener('keydown', this.handleKeyDown);
    console.log('HowToPlayPopupが表示されました'); // デバッグ用
  },
  beforeDestroy() {
    window.removeEventListener('keydown', this.handleKeyDown);
  },
  methods: {
    handleKeyDown(event) {
      if (event.key === 'ArrowLeft') {
        this.prevPage();
      } else if (event.key === 'ArrowRight') {
        this.nextPage();
      }
    }
  }
};
</script>

<style scoped>
.how-to-play-popup {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
}

.popup-content {
  position: relative;
  background-color: white;
  padding: 2rem;
  border-radius: 10px;
  max-width: 80%;
  max-height: 80%;
  overflow-y: auto;
  text-align: center;
}

.close-btn {
  position: absolute;
  top: 1rem;
  right: 1rem;
  font-size: 1.5rem;
  background: none;
  border: none;
  cursor: pointer;
}

.svg-container img {
  max-width: 100%;
  height: auto;
}

.controls {
  margin-top: 1rem;
}

.controls button {
  font-size: 1.5rem;
  padding: 0.5rem 1rem;
  margin: 0 0.5rem;
  border: none;
  border-radius: 5px;
  background-color: #ff6f61;
  color: white;
  cursor: pointer;
  transition: background-color 0.3s;
}

.controls button:hover {
  background-color: #ff3b2f;
}

.controls button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}
</style>