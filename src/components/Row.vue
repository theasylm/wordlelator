<script setup>
  import Tile from './Tile.vue'
  defineProps({
    letters: Array,
    keyboard: Boolean,
    guessNotInDictionary: Boolean,
    completed: Boolean,
    guess: Number,
    currentGuess: Number,
    currentPosition: Number,
    wordLength: Number
  })
</script>

<template>
  <div class="row" :class="{'current': guess == currentGuess, 'not-word': guessNotInDictionary && guess == currentGuess }">
    <Tile v-for="(letter, index) in letters" :letter="letter.letter" :state="letter.state" :keyboard="keyboard" :index="index" :initialized="letter.initialized" :colored="letter.colored"  :guessNotInDictionary="guessNotInDictionary" :completed="completed" :currentPosition="currentPosition" :guessNumber="guess" :currentGuess="currentGuess" :wordLength="wordLength"></Tile>
  </div>
</template>

<style>
  .row {
    display: flex;
    justify-content: center;
    margin:  .5rem 0;
  }
  .current.not-word {
    color: #842029;
  }
</style>