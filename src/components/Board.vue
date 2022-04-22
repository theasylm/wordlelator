<script setup>
  import { ref } from 'vue'
  import Row from './Row.vue'
  const props = defineProps({
    guesses: Array
  })
</script>

<template>
  <div class="board">
    <Row v-for="guess in guesses" :letters="guess"></Row>
  </div>
</template>

<style>
  .board {
    height: calc(100vh - 22.5rem);
   /**height: 10rem;**/
    overflow: scroll;
    -ms-overflow-style: none;  /* IE and Edge */
    scrollbar-width: none;  /* Firefox */
  }
  .board::-webkit-scrollbar {
    display: none;
  }
</style>