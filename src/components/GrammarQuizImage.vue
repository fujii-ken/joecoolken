<template>
  <div class="grammar-quiz">
    <h2>伝票の書き方と個数を入力してください</h2>
    <button @click="generateOrder">新しい注文を生成</button>
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
    <button @click="checkOrder" v-if="selectedItems.length > 0">注文を確認</button>
    <p v-if="feedback" :class="{ error: isError }">{{ feedback }}</p>
  </div>
</template>

<script setup>
import { ref } from 'vue';

// カフェのメニューと実際の呼び方の対応表
const wordTable = [
  { write: "桜", call: "ハム", partOfSpeech: "morning", quantity: 1, remark: "" },
  { write: "サンド", call: "白", partOfSpeech: "morning", quantity: 1, remark: "" },
  { write: "Oサンド", call: "やき", partOfSpeech: "morning", quantity: 1, remark: "" },
  { write: "マフィン", call: "マフィン", partOfSpeech: "morning", quantity: 1, remark: "" },
  { write: "B", call: "ブレンド", partOfSpeech: "setDrink", quantity: 1, remark: "" },
  { write: "S", call: "ストレート", partOfSpeech: "setDrink", quantity: 1, remark: "" },
  { write: "ダーク", call: "ダーク", partOfSpeech: "setDrink", quantity: 1, remark: "" },
  { write: "ソフト", call: "ソフト", partOfSpeech: "setDrink", quantity: 1, remark: "" },
];

// 選択されたアイテム（モーニングメニューとドリンク）
const selectedItems = ref([]);

// フィードバック
const feedback = ref("");
const isError = ref(false);

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

  feedback.value = "";
  isError.value = false;
};

// 注文を確認する関数
const checkOrder = () => {
  if (selectedItems.value.length === 0) {
    feedback.value = "注文がありません。";
    isError.value = true;
    return;
  }

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

  feedback.value = `お願いします。セットで${orderSummary}です。`;
  isError.value = false;
};
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
}

.order-table th, .order-table td {
  border: 1px solid #ccc;
  padding: 10px;
  text-align: center;
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

p {
  font-size: 18px;
  margin: 10px 0;
  white-space: pre-line; /* 改行を反映する */
}

p.error {
  color: red;
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