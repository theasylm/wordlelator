<script setup>
  import { computed } from 'vue'
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
    keyboard: Boolean,
    initialized: Boolean,
    colored: Boolean,
    guessNotInDictionary: Boolean,
    currentPosition: Number,
    currentGuess: Number,
    guessNumber: Number,
    wordLength: Number
  })

  let isCurrentTile = computed(() => {
    let effectivePosition = props.currentPosition
    if ( effectivePosition == props.wordLength ){
      effectivePosition--
    }
    return props.guessNumber == props.currentGuess && effectivePosition == props.index
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
    } else if ( props.guessNumber == props.currentGuess ) {
      window.dispatchEvent(new CustomEvent('tile-click', { detail: props.index }))
    }
  }
</script>

<template>
  <div class="tile" :class="[statusClasses[state],{ wide: letter.length > 1, revealed: state > 1 && !keyboard && !initialized, colored: colored, 'current-tile': isCurrentTile }]" @click="clickTile">
    <div class="letter">{{letter}}</div>
  </div>
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
    transition: transform 1.4s;

  }
  .current-tile {
    border-width: 5px;
  }
  .unguessed.colored {
    background-color: #818384;
  }
  .not-in-word.colored {
    background-color: #3a3a3c;
  }
  .not-in-place.colored {
    /**background-color: #e38f2f;**/
    background-color: #b99a37;
  }
  .in-place.colored {
    /**background-color: #15a858;**/
    background-color: #50834d;
  }
  .tile.wide {
    width: 7.5rem;
  }
  .tile.revealed {
    transform: rotateX(180deg);
  }
  .tile.revealed .letter {
    transform: rotateX(180deg);
    transition: transform 1.4s;
  }
  .current .tile {
    cursor: pointer;
  }
</style>
