<script setup>
  import Tile from './Tile.vue'
  defineProps({
    letters: Array
  })
</script>

<template>
  <div class="row">
    <Tile v-for="letter in letters" :letter="letter.letter" :state="letter.state"></Tile>
  </div>
</template>

<style>
  .row {
    display: flex;
    justify-content: center;
    margin:  .5rem 0;
  }
</style>