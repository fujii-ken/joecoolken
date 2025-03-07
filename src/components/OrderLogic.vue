

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

// 並び替えクイズ用のデータ
const shuffledWords = ref([]);
const correctAnswer = ref("");

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
    orderSummary += multipleQuantityDrinks.map(([call, quantity]) => `${call}${quantity}`).join(" ");
  }

  // 個数が1のドリンクを表示
  if (singleQuantityDrinks.length > 0) {
    if (orderSummary) orderSummary += " ";
    orderSummary += singleQuantityDrinks.map(([call]) => call).join(" ");
  }

  // 個数が1のドリンクがある場合、最後に「１」を追加
  if (singleQuantityDrinks.length > 0) {
    orderSummary += " １";
  }

  // orderSummaryを「商品名」「個数（数字）」に分解
  const words = [];
  const parts = orderSummary.split(" ");
  parts.forEach((part) => {
    // 商品名と個数を分離
    const match = part.match(/([^\d]+)(\d*)/);
    if (match) {
      const [_, product, quantity] = match;
      words.push(product); // 商品名
      if (quantity) words.push(quantity); // 個数
    }
  });

  // 固定フレーズを追加
  words.unshift("お願いします", "セットで");
  words.push("です。");

  // 正解の文章を生成
  correctAnswer.value = words.join(" ");

  // 単語をシャッフル
  shuffledWords.value = [...words].sort(() => Math.random() - 0.5);
};

// 外部に公開する関数やデータ
defineExpose({
  selectedItems,
  shuffledWords,
  correctAnswer,
  generateOrder,
});
</script>