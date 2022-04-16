<script setup>
  import Row from './Row.vue'
  defineProps({
    rows: Array
  })
</script>

<template>
  <div class="keyboard">
    <Row :letters="row" v-for="row in rows" :keyboard="true"></Row>
  </div>
</template>

<style>
  .keyboard {
    position: absolute;
    bottom: .5rem;
    left: 0;
    right: 0;
    margin: auto
  }
  .keyboard .tile {
    cursor: pointer;
  }
</style>