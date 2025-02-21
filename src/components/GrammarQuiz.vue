<template>
    <div class="grammar-quiz">
      <h2>{{ question }}</h2>
      <div class="words">
        <div
          v-for="(word, index) in shuffledWords"
          :key="index"
          class="word"
          @click="selectWord(word)"
        >
          {{ word }}
        </div>
      </div>
      <div class="selected-words">
        <div
          v-for="(word, index) in selectedWords"
          :key="index"
          class="word"
          @click="removeWord(index)"
        >
          {{ word }}
        </div>
      </div>
      <button @click="checkAnswer">答えを確認</button>
      <p v-if="feedback">{{ feedback }}</p>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        question: "次の単語を正しい順序に並べ替えてください。",
        words: ["お願いします", "セットで", "ブレンド", "２","です。"],
        shuffledWords: [],
        selectedWords: [],
        feedback: ""
      };
    },
    created() {
      this.shuffleWords();
    },
    methods: {
      shuffleWords() {
        this.shuffledWords = [...this.words].sort(() => Math.random() - 0.5);
      },
      selectWord(word) {
        this.selectedWords.push(word);
        this.shuffledWords = this.shuffledWords.filter(w => w !== word);
      },
      removeWord(index) {
        const word = this.selectedWords.splice(index, 1)[0];
        this.shuffledWords.push(word);
      },
      checkAnswer() {
        if (this.selectedWords.join(" ") === this.words.join(" ")) {
          this.feedback = "正解です！";
        } else {
          this.feedback = "不正解です。もう一度試してください。";
        }
      }
    }
  };
  </script>
  
  <style scoped>
  .grammar-quiz {
    text-align: center;
  }
  
  .words, .selected-words {
    display: flex;
    justify-content: center;
    margin: 20px 0;
  }
  
  .word {
    margin: 5px;
    padding: 10px;
    border: 1px solid #ccc;
    cursor: pointer;
  }
  
  .word:hover {
    background-color: #f0f0f0;
  }
  
  button {
    margin-top: 20px;
    padding: 10px 20px;
    font-size: 16px;
    cursor: pointer;
  }
  </style>