<script setup>
  import { ref, onUnmounted } from 'vue'
  import Board from './components/Board.vue'
  import Keyboard from './components/Keyboard.vue'
  import CryptoJS from 'crypto-js'
  import JSURL from 'jsurl'
  const encrypt = (text, useSalt) => {
    if ( useSalt ){
      let salt = Math.random().toString(36).slice(2, 6)
      text = salt + '||' + text
    }

    return CryptoJS.enc.Base64.stringify(CryptoJS.enc.Utf8.parse(text))
  }
  const decrypt = (data, useSalt) => {
    let word = CryptoJS.enc.Base64.parse(data).toString(CryptoJS.enc.Utf8)
    if ( useSalt ){
      return word.slice(word.indexOf('||') + 2)
    } else {
      return word
    }

  }
  const params = JSURL.parse((new URL(document.location)).searchParams.get('p'))
  const word = decrypt(params['w'],true) || ''
  const wordLength = word.length > 0 ? word.length : 5
  const numberOfGuesses = parseInt(params['g']) || 6
  const creator = params['c'] || ''
  const hint1 = params['h1'] || ''
  const hint2 = decrypt(params['h2'],false) || ''
  const msg = decrypt(params['m'],false) || ''
  let guesses = ref(Array())
  let currentGuess = ref(0)
  let givenWordCount = 0
  for ( let i=0; i < numberOfGuesses; i++ ){
    let givenWord = params['s' + (i+1)]
    if ( givenWord ) {
      givenWordCount++
    }
    let initialGuess = []
    for ( let x=0; x < wordLength; x++ ){
      let letter = ''
      if ( givenWord ) {
        letter = givenWord.charAt(x)
      }
      initialGuess.push({ 'letter': letter, 'state': 0, 'initialized': givenWord ? true : false })
    }
    guesses.value.push(initialGuess)
  }

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

  const onKey = function(key) {
    if (/^[a-zA-Z]$/.test(key)) {
      fillTile(key.toLowerCase())
    } else if (key === 'Backspace') {
      clearTile()
    } else if (key === 'Enter') {
      completeRow(false)
    }
  }

  const fillTile = function(key){
    for ( let i=0; i < guesses.value[currentGuess.value].length; i++ ){
      let tile = guesses.value[currentGuess.value][i]
      if ( tile['letter'] == '' ){
        tile['letter'] = key
        break
      }
    }
  }

  const clearTile = function() {
    for ( let i=guesses.value[currentGuess.value].length - 1; i > -1 ; i-- ){
      let tile = guesses.value[currentGuess.value][i]
      if ( tile['letter'] != '' ){
        tile['letter'] = ''
        break
      }
    }
  }

  const completeRow = function(skipAnimation) {
    let skip = skipAnimation ? 0 : 1
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
        }, 500 * (i+1) * skip)
      }
    }

    for ( let i=0; i < guess.length; i++ ) {
      if ( !changedLetters.includes(i) && answerLetters.includes(guess[i]['letter']) ){
        answerLetters[answerLetters.indexOf(guess[i]['letter'])] = null
        changedLetters.push(i)
        keyboardUpdates.push([guess[i]['letter'],3])
        setTimeout( () => {
          guess[i]['state'] = 3
        }, 500 * (i+1) * skip)
      }
    }

    for ( let i=0; i < guess.length; i++ ) {
      if ( !changedLetters.includes(i) ) {
        keyboardUpdates.push([guess[i]['letter'],2])
        setTimeout( () => {
          guess[i]['state'] = 2
        }, 500 * (i+1) * skip)
      }
    }

    setTimeout( () => {
      for ( let i=0; i < keyboardUpdates.length; i++ ) {
        updateKeyboard(keyboardUpdates[i][0],keyboardUpdates[i][1])
      }
    }, 600 * guess.length * skip)

    currentGuess.value++
  }

  const updateKeyboard = function(letter,state) {
    let keyPosition = findKey(letter)
    if ( keyboardRows.value[keyPosition[0]][keyPosition[1]]['state'] < state ){
      keyboardRows.value[keyPosition[0]][keyPosition[1]]['state'] = state
    }
  }

  const findKey = function(letter) {
    for ( let i=0; i < 3; i++ ) {
      for ( let x=0; x < keyboardRows.value[i].length; x++ ){
        if ( letter === keyboardRows.value[i][x]['letter'] ) {
          return [i,x]
        }
      }
    }
  }

  for ( let i=0; i < givenWordCount; i++ ){
    completeRow(true)
  }
</script>

<template>
  <h1>Wordlelator</h1>
  <div class="info">
    <span class="creator" v-if="creator != ''">Creator: {{creator}}</span><br/>
    <span class="hint1" v-if="hint1 != ''">Hint: {{hint1}}</span><br/>
    <span class="hint2" v-if="hint2 != ''">Hint2: {{hint2}}</span><br/>
    <span class="msg" v-if="msg != ''">Message: {{msg}}</span><br/>
    <span >guess: {{currentGuess}}</span><br/>
  </div>
  <h3>{{JSURL.stringify({
    'w': encrypt('testing',true),
    'c': 'theasylm',
    'g': 9,
    'h1': 'short hint',
    'h2': encrypt('very very long hint. oh so very long hint. this hint just goes on and on and on.',false),
    'm': encrypt('Congrats!',false),
    's1': 'attests',
    's2': 'doggoes'
  })}}</h3>
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
    margin: 0;
  }
</style>
