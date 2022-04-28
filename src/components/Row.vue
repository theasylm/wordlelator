<script setup>
  import Tile from './Tile.vue'
  defineProps({
    letters: Array,
    keyboard: Boolean,
    guessNotInDictionary: Boolean
  })
</script>

<template>
  <div class="row">
    <Tile v-for="(letter, index) in letters" :letter="letter.letter" :state="letter.state" :keyboard="keyboard" :index="index" :initialized="letter.initialized" :colored="letter.colored"  :guessNotInDictionary="guessNotInDictionary"></Tile>
  </div>
</template>

<style>
  .row {
    display: flex;
    justify-content: center;
    margin:  .5rem 0;
  }
</style>