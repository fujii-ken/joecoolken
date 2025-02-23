<script setup>
import HelloWorld from './components/HelloWorld.vue'
import GrammarQuiz from './components/GrammarQuiz.vue';
import GrammarQuizImage from './components/GrammarQuizImage.vue';
import GrammarQuizImageDetail from './components/GrammarQuizImageDetail.vue';
</script>

<template>
    <div class="components">
      <!-- <HelloWorld msg="文法並び替えアプリ" /> -->
      <GrammarQuizImage />
      <!-- <GrammarQuizImageDetail/> -->
    </div>
</template>

<style scoped>
.components{
  background-color:#F4EEEE;
}
</style>

