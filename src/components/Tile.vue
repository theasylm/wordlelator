<script setup>
  import { ref } from 'vue'
  const Empty = 0
  const Unguessed = 1
  const NotInWord = 2
  const NotInPlace = 3
  const InPlace = 4
  const statusClasses = ['empty','unguessed','not-in-word','not-in-place','in-place']
  const props = defineProps({
    state: {
      type: Number,
      default: 0
    },
    letter: {
     type: String,
     default: ''
    },
    index: Number,
    keyboard: Boolean
  })

  let clickTile = function(){
    if ( props.keyboard ) {
      let letter = props.letter
      if ( letter === 'enter' ) {
        letter = 'Enter'
      } else if ( letter === 'del' ){
        letter = 'Backspace'
      }
      window.dispatchEvent(new KeyboardEvent('keyup', {'key': letter}))
    }
  }
</script>

<template>
  <div class="tile" :class="[statusClasses[state],{ wide: letter.length > 1, revealed: state > 1 && !keyboard }]" @click="clickTile" :style="{ transitionDelay: `${keyboard ? 0 : index * 0}ms` }">{{letter}}</div>
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
    transition: transform .8s;

  }
  .unguessed {
    background-color: #818384;
  }
  .not-in-word {
    background-color: #3a3a3c;
  }
  .not-in-place {
    background-color: #e38f2f;
  }
  .in-place {
    background-color: #15a858;
  }
  .tile.wide {
    width: 7.5rem;
  }
  .tile.revealed {
    transform: rotateX(360deg);
  }
</style>
