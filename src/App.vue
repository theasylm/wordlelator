<script setup>
  import { ref, onUnmounted } from 'vue'
  import Board from './components/Board.vue'
  import Keyboard from './components/Keyboard.vue'
  const params = (new URL(document.location)).searchParams;
  const word = params.get('word') || ''
  const wordLength = word.length > 0 ? word.length : 5
  const numberOfGuesses = parseInt(params.get('guesses')) || 6
  let guesses = ref(Array())
  for ( let i=0; i < numberOfGuesses; i++ ){
    let initialGuess = []
    for ( let x=0; x < wordLength; x++ ){
      initialGuess.push({ 'letter': '', 'state': 0})
    }
    guesses.value.push(initialGuess)
  }
  let currentGuess = ref(0)
  let keyboardRows = [
    [
      {
        'letter': 'q',
        'state': 4
      },
      {
        'letter': 'w',
        'state': 4
      },
      {
        'letter': 'e',
        'state': 4
      },
      {
        'letter': 'r',
        'state': 4
      },
      {
        'letter': 't',
        'state': 4
      },
      {
        'letter': 'y',
        'state': 4
      },
      {
        'letter': 'u',
        'state': 4
      },
      {
        'letter': 'i',
        'state': 4
      },
      {
        'letter': 'o',
        'state': 4
      },
      {
        'letter': 'p',
        'state': 4
      }
    ],
    [
      {
        'letter': 'a',
        'state': 4
      },
      {
        'letter': 's',
        'state': 4
      },
      {
        'letter': 'd',
        'state': 4
      },
      {
        'letter': 'f',
        'state': 4
      },
      {
        'letter': 'g',
        'state': 4
      },
      {
        'letter': 'h',
        'state': 4
      },
      {
        'letter': 'j',
        'state': 4
      },
      {
        'letter': 'k',
        'state': 4
      },
      {
        'letter': 'l',
        'state': 4
      }
    ],
    [
      {
        'letter': 'enter',
        'state': 4
      },
      {
        'letter': 'z',
        'state': 4
      },
      {
        'letter': 'x',
        'state': 4
      },
      {
        'letter': 'c',
        'state': 4
      },
      {
        'letter': 'v',
        'state': 4
      },
      {
        'letter': 'b',
        'state': 4
      },
      {
        'letter': 'n',
        'state': 4
      },
      {
        'letter': 'm',
        'state': 4
      },
      {
        'letter': 'del',
        'state': 4
      }
    ]
  ]

  const onKeyup = (e) => onKey(e.key)
  window.addEventListener('keyup', onKeyup)
  onUnmounted(() => {
    window.removeEventListener('keyup', onKeyup)
  })

  let onKey = function(key) {
    if (/^[a-zA-Z]$/.test(key)) {
      fillTile(key.toLowerCase())
    } else if (key === 'Backspace') {
      clearTile()
    } else if (key === 'Enter') {
      //completeRow()
    }
  }

  let fillTile = function(key){
    for ( let i=0; i < guesses.value[currentGuess.value].length; i++ ){
      let tile = guesses.value[currentGuess.value][i]
      if ( tile['letter'] == '' ){
        tile['letter'] = key
        break
      }
    }
  }

  let clearTile = function() {
    for ( let i=guesses.value[currentGuess.value].length - 1; i > -1 ; i-- ){
      let tile = guesses.value[currentGuess.value][i]
      if ( tile['letter'] != '' ){
        tile['letter'] = ''
        break
      }
    }
  }
</script>

<template>
  <h1>Wordlelator</h1>
  <Board :guesses="guesses"></Board>
  <Keyboard :rows="keyboardRows"></Keyboard>
</template>

<style>
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #efefef;
    background-color: #08111b;
    height: 100vh;
    padding-top: 1em;
  }

  h1 {
    margin-top: 0;
  }
</style>
