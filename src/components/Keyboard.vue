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
    bottom: 2rem;
    left: 0;
    right: 0;
    margin: auto
  }
  .keyboard .tile {
    cursor: pointer;
  }
  @media (max-width: 766px){
    .keyboard .tile {
      width:  2.5rem;
      height: 2.5rem;
      font-size: 2rem;
    }
    .keyboard .tile.wide {
      width: 5.5rem;
    }

  }

  @media (max-width: 500px){
    .keyboard .tile {
      width:  1.5rem;
      height: 1.5rem;
      font-size: 1.25rem;
    }
    .keyboard .tile.wide {
      width: 4rem;
    }

  }
  @media (max-width: 400px){
    .keyboard .tile {
      width:  1rem;
      height: 1.5rem;
      font-size: .75rem;
    }
    .keyboard .tile.wide {
      width: 2.25rem;
    }

  }
</style>