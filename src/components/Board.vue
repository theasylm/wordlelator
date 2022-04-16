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
    height: calc(100vh - 19.5rem);
    overflow: scroll;
  }
</style>