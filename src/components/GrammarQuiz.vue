<template>
    <div class="grammar-quiz">
      <h2>{{ question }}</h2>
      <div class="difficulty-selector">
        <label for="difficulty">難易度:</label>
        <select id="difficulty" v-model="selectedDifficulty" @change="generateNewSentence">
          <option value="easy">初級</option>
          <option value="medium">中級</option>
          <option value="hard">上級</option>
        </select>
      </div>
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
      <p v-if="showAnswer">正解: {{ correctAnswer }}</p>
      <button @click="generateNewSentence" v-if="isAnswerChecked">次の問題</button>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted } from 'vue';
  
  // 文章生成のための単語リスト
  const mornings = ["", "You", "He", "She", "We", "They"];
  const drinks = ["ブレンド", "ストレート", "ダーク", "アイスティ", "レーコー", "ホットティ"];
  const calls = ["お願いします"];
  const types = ["単品で", "セットで"];
  const amounts = ["ワンです。", "ツーです。", "スリーです。", "四です。"];
  
  // データの定義
  const question = ref("");
  const words = ref([]);
  const shuffledWords = ref([]);
  const selectedWords = ref([]);
  const feedback = ref("");
  const isAnswerChecked = ref(false);
  const showAnswer = ref(false);
  const correctAnswer = ref("");
  const selectedDifficulty = ref("easy");
  
  // 難易度に基づいて文章を生成する関数
  const generateSentence = (difficulty) => {
    const morning = mornings[Math.floor(Math.random() * mornings.length)];
    const drink = drinks[Math.floor(Math.random() * drinks.length)];
    const call = calls[Math.floor(Math.random() * calls.length)];
    const amount = amounts[Math.floor(Math.random() * amounts.length)];
    const type = types[Math.floor(Math.random() * types.length)];
  
    let sentence = "";
  
    switch (difficulty) {
      case "easy":
        sentence = `${call} ${type} ${drink} ${amount}`;
        break;
      case "medium":
        sentence = `${morning} ${amount} ${drink} ${call}`;
        break;
      case "hard":
        const secondmorning = mornings[Math.floor(Math.random() * mornings.length)];
        const seconddrink = drinks[Math.floor(Math.random() * drinks.length)];
        const secondcall = calls[Math.floor(Math.random() * calls.length)];
        sentence = `${morning} ${drink} ${call} ${type} ${secondmorning} ${seconddrink} ${secondcall}`;
        break;
      default:
        sentence = `${morning} ${drink} ${call}`;
    }
  
    return sentence;
  };
  
  // 文章を単語に分割する関数
  const splitSentenceIntoWords = (sentence) => {
    return sentence.split(" ");
  };
  
  // 単語をシャッフルする関数
  const shuffleWords = () => {
    shuffledWords.value = [...words.value].sort(() => Math.random() - 0.5);
    selectedWords.value = [];
    feedback.value = "";
    showAnswer.value = false;
    isAnswerChecked.value = false;
  };
  
  // 単語を選択する関数
  const selectWord = (word) => {
    selectedWords.value.push(word);
    shuffledWords.value = shuffledWords.value.filter(w => w !== word);
  };
  
  // 単語を削除する関数
  const removeWord = (index) => {
    const word = selectedWords.value.splice(index, 1)[0];
    shuffledWords.value.push(word);
  };
  
  // 答えを確認する関数
  const checkAnswer = () => {
    if (selectedWords.value.join(" ") === words.value.join(" ")) {
      feedback.value = "正解です！";
    } else {
      feedback.value = "不正解です。もう一度試してください。";
    }
    correctAnswer.value = words.value.join(" ");
    showAnswer.value = true;
    isAnswerChecked.value = true;
  };
  
  // 新しい問題を生成する関数
  const generateNewSentence = () => {
    const sentence = generateSentence(selectedDifficulty.value);
    question.value = "次の単語を正しい順序に並べ替えてください。";
    words.value = splitSentenceIntoWords(sentence);
    correctAnswer.value = sentence;
    shuffleWords();
  };
  
  // コンポーネントがマウントされたときに最初の問題を生成
  onMounted(() => {
    generateNewSentence();
  });
  </script>
  
  <style scoped>
  .grammar-quiz {
    text-align: center;
  }
  
  .difficulty-selector {
    margin-bottom: 20px;
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