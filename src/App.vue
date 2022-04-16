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
  let keyboardRows = ref([
    [
      {
        'letter': 'q',
        'state': 1
      },
      {
        'letter': 'w',
        'state': 1
      },
      {
        'letter': 'e',
        'state': 1
      },
      {
        'letter': 'r',
        'state': 1
      },
      {
        'letter': 't',
        'state': 1
      },
      {
        'letter': 'y',
        'state': 1
      },
      {
        'letter': 'u',
        'state': 1
      },
      {
        'letter': 'i',
        'state': 1
      },
      {
        'letter': 'o',
        'state': 1
      },
      {
        'letter': 'p',
        'state': 1
      }
    ],
    [
      {
        'letter': 'a',
        'state': 1
      },
      {
        'letter': 's',
        'state': 1
      },
      {
        'letter': 'd',
        'state': 1
      },
      {
        'letter': 'f',
        'state': 1
      },
      {
        'letter': 'g',
        'state': 1
      },
      {
        'letter': 'h',
        'state': 1
      },
      {
        'letter': 'j',
        'state': 1
      },
      {
        'letter': 'k',
        'state': 1
      },
      {
        'letter': 'l',
        'state': 1
      }
    ],
    [
      {
        'letter': 'enter',
        'state': 1
      },
      {
        'letter': 'z',
        'state': 1
      },
      {
        'letter': 'x',
        'state': 1
      },
      {
        'letter': 'c',
        'state': 1
      },
      {
        'letter': 'v',
        'state': 1
      },
      {
        'letter': 'b',
        'state': 1
      },
      {
        'letter': 'n',
        'state': 1
      },
      {
        'letter': 'm',
        'state': 1
      },
      {
        'letter': 'del',
        'state': 1
      }
    ]
  ])

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
      completeRow()
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

  let completeRow = function() {
    let guess = guesses.value[currentGuess.value]
    let i = guess.length - 1
    if ( guess[i]['letter'] === '' ) {
      return
    }

    let answerLetters = word.split('')
    let changedLetters = []
    let keyboardUpdates = []
    for ( let i=0; i < guess.length; i++ ) {
      if ( guess[i]['letter'] === answerLetters[i] ) {
        answerLetters[i] = null
        changedLetters.push(i)
        keyboardUpdates.push([guess[i]['letter'],4])
        setTimeout( () => {
          guess[i]['state'] = 4
        }, 500 * (i+1))
      }
    }

    for ( let i=0; i < guess.length; i++ ) {
      if ( !changedLetters.includes(i) && answerLetters.includes(guess[i]['letter']) ){
        answerLetters[answerLetters.indexOf(guess[i]['letter'])] = null
        changedLetters.push(i)
        keyboardUpdates.push([guess[i]['letter'],3])
        setTimeout( () => {
          guess[i]['state'] = 3
        }, 500 * (i+1))
      }
    }

    for ( let i=0; i < guess.length; i++ ) {
      if ( !changedLetters.includes(i) ) {
        keyboardUpdates.push([guess[i]['letter'],2])
        setTimeout( () => {
          guess[i]['state'] = 2
        }, 500 * (i+1))
      }
    }

    setTimeout( () => {
      for ( let i=0; i < keyboardUpdates.length; i++ ) {
        updateKeyboard(keyboardUpdates[i][0],keyboardUpdates[i][1])
      }
    }, 600 * guess.length)

    currentGuess.value++
  }

  let updateKeyboard = function(letter,state) {
    let keyPosition = findKey(letter)
    console.log(keyPosition)
    if ( keyboardRows.value[keyPosition[0]][keyPosition[1]]['state'] < state ){
      keyboardRows.value[keyPosition[0]][keyPosition[1]]['state'] = state
    }
  }

  let findKey = function(letter) {
    for ( let i=0; i < 3; i++ ) {
      console.log(keyboardRows.value[i][0])
      for ( let x=0; x < keyboardRows.value[i].length; x++ ){
        if ( letter === keyboardRows.value[i][x]['letter'] ) {
          return [i,x]
        }
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
    /**background-color: #08111b;**/
    background-color: #011637;
    height: 100vh;
    padding-top: 1em;
  }

  h1 {
    margin-top: 0;
  }
</style>
