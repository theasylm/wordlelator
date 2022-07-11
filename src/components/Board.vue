<script setup>
  import Row from './Row.vue'
  const props = defineProps({
    guesses: Array,
    currentGuess: Number,
    guessNotInDictionary: Boolean,
    currentPosition: Number,
    wordLength: Number,
    hasTitle: Boolean
  })
</script>

<template>
  <div class="board" id="board" :class="{'has-title': hasTitle}">
    <Row v-for="guess,i in guesses" :letters="guess"  :completed="guess['completed']" :guess="i" :currentGuess="currentGuess" :guessNotInDictionary="guessNotInDictionary" :currentPosition="currentPosition" :wordLength="wordLength"></Row>
  </div>
</template>

<style>
  .board {
    height: calc(100vh - 22rem);
    overflow: scroll;
    -ms-overflow-style: none;  /* IE and Edge */
    scrollbar-width: none;  /* Firefox */
  }
  .board::-webkit-scrollbar {
    display: none;
  }
  .board.has-title {
    height: calc(100vh - 24.5rem);
  }
</style>