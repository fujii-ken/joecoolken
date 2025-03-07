<template>
  <div class="grammar-quiz">
    <h2>伝票の書き方と個数を入力してください</h2>
    <button @click="startCountdown" v-if="!isGameStarted && !isCountdownActive && !isGameEnded">スタート</button>

    <!-- カウントダウン表示 -->
    <div v-if="isCountdownActive" class="countdown-overlay">
      <div class="countdown">
        <p>開始まで: {{ countdown }}秒</p>
      </div>
    </div>

    <!-- タイマー表示 -->
    <div v-if="isGameStarted && !isGameEnded" class="timer">
      <p>残り時間: {{ timeLeft }}秒</p>
      <button @click="togglePause" class="pause-button">
        {{ isPaused ? '再開' : '一時停止' }}
      </button>
    </div>

    <!-- 伝票テーブル -->
    <table class="order-table" v-if="selectedItems.length > 0">
      <thead>
        <tr>
          <th>伝票の書き方 (write)</th>
          <th>個数</th>
          <th>備考</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, index) in selectedItems" :key="index">
          <td>{{ item.write }}</td>
          <td>{{ item.quantity }}</td>
          <td>{{ item.remark }}</td>
        </tr>
      </tbody>
    </table>

    <!-- 並び替えクイズ -->
    <div v-if="isGameStarted && !isGameEnded" class="quiz-section">
      <h3>以下の単語を正しい順序に並べ替えてください</h3>
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
          class="word selected"
          @click="removeWord(index)"
        >
          {{ word }}
        </div>
      </div>
      <!-- フィードバックと正解表示 -->
      <p v-if="feedback" :class="{ error: isError }">{{ feedback }}</p>
      <p v-if="showAnswer">正解: {{ correctAnswer }}</p>
    </div>

    <!-- ゲーム終了時のスコア表示 -->
    <div v-if="isGameEnded" class="score-section">
      <h3>ゲーム終了！</h3>
      <p>正解数: {{ correctCount }}</p>
      <p>不正解数: {{ incorrectCount }}</p>
      <p>スコア: {{ score }}</p>
      <p class="comment">{{ comment }}</p>
      <button @click="restartGame">もう一度プレイ</button>
    </div>
  </div>
</template>

<script setup>


// カフェのメニューと実際の呼び方の対応表
const wordTable = [
  { write: "桜", call: "ハム", partOfSpeech: "morning", quantity: 1, remark: "" },
  { write: "サンド", call: "白", partOfSpeech: "morning", quantity: 1, remark: "" },
  { write: "Oサンド", call: "やき", partOfSpeech: "morning", quantity: 1, remark: "" },
  { write: "マフィン", call: "マフィン", partOfSpeech: "morning", quantity: 1, remark: "" },
  { write: "B", call: "ブレンド", partOfSpeech: "setDrink", quantity: 1, remark: "" },
  { write: "S", call: "ストレート", partOfSpeech: "setDrink", quantity: 1, remark: "" },
  // { write: "ダーク", call: "ダーク", partOfSpeech: "setDrink", quantity: 1, remark: "" },
  // { write: "ソフト", call: "ソフト", partOfSpeech: "setDrink", quantity: 1, remark: "" },
];


import { ref, watch } from 'vue';
// ゲーム状態
const isGameStarted = ref(false);
const isGameEnded = ref(false);
const isCountdownActive = ref(false);
const countdown = ref(3);
const timeLeft = ref(90); // 制限時間（秒）
const correctCount = ref(0);
const incorrectCount = ref(0);
const score = ref(0);
const comment = ref("");
const isPaused = ref(false);
let timerInterval = null;

// 選択されたアイテム（モーニングメニューとドリンク）
const selectedItems = ref([]);

// 並び替えクイズ用のデータ
const shuffledWords = ref([]);
const selectedWords = ref([]);
const feedback = ref("");
const isError = ref(false);
const showAnswer = ref(false);
const correctAnswer = ref("");

// カウントダウンを開始する関数
const startCountdown = () => {
  isCountdownActive.value = true;
  const interval = setInterval(() => {
    countdown.value--;
    if (countdown.value === 0) {
      clearInterval(interval);
      isCountdownActive.value = false;
      startGame();
    }
  }, 1000);
};

// ゲームを開始する関数
const startGame = () => {
  isGameStarted.value = true;
  isGameEnded.value = false;
  timeLeft.value = 90; // 制限時間を90秒に設定
  correctCount.value = 0;
  incorrectCount.value = 0;
  score.value = 0;
  generateOrder();

  // タイマーを開始
  startTimer();
};

// タイマーを開始する関数
const startTimer = () => {
  timerInterval = setInterval(() => {
    if (!isPaused.value) {
      timeLeft.value--;
      if (timeLeft.value === 0) {
        clearInterval(timerInterval);
        endGame();
      }
    }
  }, 1000);
};

// 一時停止を切り替える関数
const togglePause = () => {
  isPaused.value = !isPaused.value;
};

// ゲームを終了する関数
const endGame = () => {
  isGameEnded.value = true;
  isGameStarted.value = false;
  calculateScore();
  showComment();
};

// スコアを計算する関数
const calculateScore = () => {
  score.value = correctCount.value * 100 - incorrectCount.value * 50;
};

// コメントを表示する関数
const showComment = () => {
  if (score.value >= 300) {
    comment.value = "素晴らしい！完璧です！";
  } else if (score.value >= 200) {
    comment.value = "いい感じです！もう少し頑張りましょう！";
  } else if (score.value >= 100) {
    comment.value = "まずまずです！次はもっと頑張りましょう！";
  } else {
    comment.value = "もう一度挑戦してみましょう！";
  }
};

// ゲームをリスタートする関数
const restartGame = () => {
  isGameEnded.value = false;
  isGameStarted.value = false;
  isCountdownActive.value = false;
  countdown.value = 3;
  selectedItems.value = [];
  shuffledWords.value = [];
  selectedWords.value = [];
  feedback.value = "";
  showAnswer.value = false;
  correctAnswer.value = "";
  isPaused.value = false;

  // 2回目以降はカウントダウンなしでゲームを開始
  startGame();
};

// ランダムでモーニングメニューとドリンクを選択する関数
const generateOrder = () => {
  const morningItems = wordTable.filter(item => item.partOfSpeech === "morning");
  const setDrinkItems = wordTable.filter(item => item.partOfSpeech === "setDrink");

  // モーニングメニューをランダムで1〜4つ選択
  const selectedMorning = [];
  const maxMorningItems = Math.min(4, morningItems.length); // 最大4つに変更
  const numMorningItems = Math.floor(Math.random() * maxMorningItems) + 1;

  for (let i = 0; i < numMorningItems; i++) {
    const randomIndex = Math.floor(Math.random() * morningItems.length);
    selectedMorning.push({ ...morningItems[randomIndex] });
  }

  // 各モーニングメニューに対応するドリンクをランダムで選択
  const selectedDrinks = selectedMorning.map(() => {
    const randomIndex = Math.floor(Math.random() * setDrinkItems.length);
    return { ...setDrinkItems[randomIndex] };
  });

  // 同じモーニングメニューをまとめる
  const groupedItems = [];
  const morningCounts = {};

  selectedMorning.forEach((morning, index) => {
    if (!morningCounts[morning.write]) {
      morningCounts[morning.write] = {
        morning: { ...morning, quantity: 1 },
        drinks: [selectedDrinks[index]],
      };
    } else {
      morningCounts[morning.write].morning.quantity += 1;
      morningCounts[morning.write].drinks.push(selectedDrinks[index]);
    }
  });

  // 選択されたアイテムを伝票用に整形
  selectedItems.value = [];
  Object.values(morningCounts).forEach((group, index) => {
    selectedItems.value.push(group.morning);

    // ドリンクの個数をまとめる
    const drinkCounts = {};
    group.drinks.forEach(drink => {
      if (!drinkCounts[drink.write]) {
        drinkCounts[drink.write] = { ...drink, quantity: 1 };
      } else {
        drinkCounts[drink.write].quantity += 1;
      }
    });

    // ドリンクを伝票に追加
    Object.values(drinkCounts).forEach(drink => {
      selectedItems.value.push(drink);
    });

    if (index < Object.keys(morningCounts).length - 1) {
      selectedItems.value.push({ write: "", quantity: "", remark: "" }); // 1行開ける
    }
  });

  // 並び替えクイズ用のデータを生成
  generateQuiz();
};

// 並び替えクイズを生成する関数
const generateQuiz = () => {
  // ドリンクの個数をまとめる
  const drinkSummary = {};
  selectedItems.value
    .filter(item => item.partOfSpeech === "setDrink")
    .forEach(drink => {
      if (!drinkSummary[drink.call]) {
        drinkSummary[drink.call] = drink.quantity;
      } else {
        drinkSummary[drink.call] += drink.quantity;
      }
    });

  // ドリンクの個数を独自のルールで表示
  const drinksWithQuantity = Object.entries(drinkSummary);

  // 個数が2以上のドリンクと1のドリンクを分ける
  const multipleQuantityDrinks = drinksWithQuantity.filter(([_, quantity]) => quantity > 1);
  const singleQuantityDrinks = drinksWithQuantity.filter(([_, quantity]) => quantity === 1);

  let orderSummary = "";

  // 個数が2以上のドリンクをそのまま表示
  if (multipleQuantityDrinks.length > 0) {
    orderSummary += multipleQuantityDrinks.map(([call, quantity]) => `${call}${quantity}`).join("、");
  }

  // 個数が1のドリンクを表示
  if (singleQuantityDrinks.length > 0) {
    if (orderSummary) orderSummary += "、";
    orderSummary += singleQuantityDrinks.map(([call]) => call).join("、");
  }

  // 個数が1のドリンクがある場合、最後に「１」を追加
  if (singleQuantityDrinks.length > 0) {
    orderSummary += "１";
  }

  // orderSummaryを「商品名」「個数（数字）」に分解
  const words = [];
  const parts = orderSummary.split("、");
  parts.forEach((part) => {
    // 商品名と個数を分離
    const match = part.match(/([^\d]+)(\d*)/);
    if (match) {
      const [_, product, quantity] = match;
      words.push(product); // 商品名
      if (quantity) {
        // 個数をドリンクの数だけ独立して追加
        words.push(quantity);
      }
    }
  });

  // 固定フレーズを追加
  words.unshift("お願いします", "セットで");
  words.push("です。");

  // 正解の文章を生成
  correctAnswer.value = words.join(" ");

  // 単語をシャッフル
  shuffledWords.value = [...words].sort(() => Math.random() - 0.5);
  selectedWords.value = [];
  feedback.value = "";
  showAnswer.value = false;
};

// 単語を選択する関数（修正）
const selectWord = (word) => {
  // 単語が既に選択されている場合は早期リターン
  if (selectedWords.value.includes(word)) {
    return;
  }
  
  // 選択された単語を追加
  selectedWords.value.push(word);

  // 選択された単語を選択肢から削除
  const index = shuffledWords.value.indexOf(word);
  if (index !== -1) {
    shuffledWords.value.splice(index, 1);
  }

  // すべての単語を選択したら自動的に答えをチェック
  if (selectedWords.value.length === correctAnswer.value.split(" ").length) {
    checkAnswer();
  }
};

// 単語を削除する関数（修正）
const removeWord = (indexOrWord) => {
  let index;
  if (typeof indexOrWord === 'number') {
    // インデックスが渡された場合
    index = indexOrWord;
  } else {
    // 単語が渡された場合
    index = selectedWords.value.indexOf(indexOrWord);
  }
  
  if (index !== -1) {
    const removedWord = selectedWords.value[index];
    selectedWords.value.splice(index, 1);
    shuffledWords.value.push(removedWord); // 選択解除された単語を選択肢に戻す
    feedback.value = ""; // フィードバックをリセット
    showAnswer.value = false; // 正解表示をリセット
  }
};
// 答えを確認する関数
const checkAnswer = () => {
  if (selectedWords.value.join(" ") === correctAnswer.value) {
    feedback.value = "正解です！";
    isError.value = false;
    correctCount.value++;
  } else {
    feedback.value = "不正解です。もう一度試してください。";
    isError.value = true;
    incorrectCount.value++;
  }
  showAnswer.value = true;

  // 次の問題を生成
  setTimeout(() => {
    generateOrder();
    feedback.value = ""; // フィードバックをリセット
    showAnswer.value = false; // 正解表示をリセット
  }, 1500); // 1.5秒後に次の問題を生成
};
// selectedWordsを監視して自動的に答えをチェック
watch(selectedWords, (newValue) => {
  if (newValue.length === correctAnswer.value.split(" ").length) {
    checkAnswer();
  }
});
</script>

<style scoped>
.grammar-quiz {
  text-align: center;
  padding: 10px;
}

.order-table {
  width: 100%;
  border-collapse: collapse;
  margin: 20px auto;
  background-color: white; /* 表の中身を白色に */
}

.order-table th, .order-table td {
  border: 1px solid #ccc;
  padding: 10px;
  text-align: center;
  background-color: white; /* セルの背景色を白色に */
}

button {
  margin-top: 20px;
  padding: 15px 30px;
  font-size: 18px;
  cursor: pointer;
  border: none;
  background-color: #007bff;
  color: white;
  border-radius: 5px;
}

button:hover {
  background-color: #0056b3;
}

.pause-button {
  margin-top: 10px;
  background-color: #ffc107;
  color: black;
}

.pause-button:hover {
  background-color: #e0a800;
}

.next-button {
  margin-top: 20px;
  background-color: #28a745;
}

.next-button:hover {
  background-color: #218838;
}

.quiz-section {
  margin-top: 20px;
}

.words, .selected-words {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  margin: 20px 0;
}

.selected-words .word.selected {
  background-color: pink; /* 選択された単語をピンク色に */
}

.word {
  margin: 5px;
  padding: 10px;
  border: 1px solid #ccc;
  cursor: pointer;
  font-size: 18px;
}

.word:hover {
  background-color: #f0f0f0;
}


p {
  font-size: 18px;
  margin: 10px 0;
  white-space: pre-line; /* 改行を反映する */
}

p.error {
  color: red;
}

.countdown-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.countdown {
  font-size: 48px;
  font-weight: bold;
  color: white;
}

.timer {
  font-size: 24px;
  font-weight: bold;
  margin: 20px 0;
}

.score-section {
  margin-top: 20px;
}

.comment {
  font-size: 20px;
  font-weight: bold;
  color: #28a745;
}

/* スマートフォン用のスタイル */
@media (max-width: 600px) {
  .order-table th, .order-table td {
    padding: 8px;
    font-size: 14px;
  }

  button {
    padding: 12px 24px;
    font-size: 16px;
  }

  p {
    font-size: 16px;
  }
}
</style>