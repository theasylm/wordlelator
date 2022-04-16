<script setup>
  import Tile from './Tile.vue'
  defineProps({
    letters: Array,
    keyboard: Boolean
  })
</script>

<template>
  <div class="row">
    <Tile v-for="(letter, index) in letters" :letter="letter.letter" :state="letter.state" :keyboard="keyboard" :index="index"></Tile>
  </div>
</template>

<style>
  .row {
    display: flex;
    justify-content: center;
    margin:  .5rem 0;
  }
</style>