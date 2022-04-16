<script setup>
  import { ref } from 'vue'
  const Empty = 0
  const NotInWord = 1
  const NotInPlace = 2
  const InPlace = 3
  const Unguessed = 4
  const statusClasses = ['empty','not-in-word','not-in-place','in-place','unguessed']
  defineProps({
    state: {
      type: Number,
      default: 0
    },
    letter: {
     type: String,
     default: ''
    }
  })

</script>

<template>
  <div class="tile" :class="[statusClasses[state],{ wide: letter.length > 1 }]">{{letter}}</div>
</template>

<style>
  .tile {
    width: 3rem;
    height: 3rem;
    border: 2px solid #efefef;
    margin-right: .5rem;
    border-radius: .25rem;
    text-transform: uppercase;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 2rem;
    font-weight: 600;
  }
  .unguessed {
    background-color: #434d5d;
  }
  .tile.wide {
    width: 7.5rem;
  }
</style>
