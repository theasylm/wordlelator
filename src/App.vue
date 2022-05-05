<script setup>
  import { $vfm, VueFinalModal, ModalsContainer } from 'vue-final-modal'
  import { ref, onUnmounted, onMounted, computed } from 'vue'
  import Board from './components/Board.vue'
  import Keyboard from './components/Keyboard.vue'
  import CryptoJS from 'crypto-js'
  import JSURL from 'jsurl'
  import { PencilIcon, QuestionMarkCircleIcon, LightBulbIcon, XIcon, ChartBarIcon, RefreshIcon } from '@heroicons/vue/outline'
  import { allWords } from './assets/js/allWords.js'
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
  let showFormModal = function() {
    showModal.value = true
    setTimeout(() => {document.getElementById('word').focus()},350)
  }
  let params = (new URL(document.location)).searchParams.get('p')
  if ( params ) {
    if ( !params.match(/\)$/) ){
      params += ")"
    }
    params = JSURL.parse(params)
  }
  let word = ''
  let showModal = ref(false)
  if ( params ){
    word = decrypt(params['w'],true)
  } else {
    showFormModal()
  }
  const wordLength = word.length > 0 ? word.length : 0
  const allPossibleWords = allWords[wordLength]
  let numberOfGuesses = 0
  let creator = ''
  let hint1 = ''
  let hint2 = ''
  let msg = ref('')
  let revealStartingLetter = false
  if ( params ){
    numberOfGuesses = parseInt(params['g'])
    creator = params['c'] || ''
    hint1 = params['h1'] || ''
    hint2 = params['h2'] || ''
    if ( hint2 != '' ) {
      hint2 = decrypt(hint2,false)
    }
    revealStartingLetter = params['r'] == 1 ? true : false
    msg.value = params['m'] || ''
  }

  if ( msg.value != '' ) {
    msg.value = decrypt(msg.value,false)
  } else {
    msg.value = 'Nice!'
  }
  let newWord = ref('')
  let newCreator = ref('')
  let newNumberOfGuesses = ref(6)
  let newHint1 = ref('')
  let newHint2 = ref('')
  let newMessage = ref('')
  let newStartingWords = ref(Array())
  let newRevealStartingLetter = ref(false)
  let newUrl = ref('')
  let showHintModal = ref(false)
  let showWinModal = ref(false)
  let showHelpModal = ref(false)
  let showConfirmModal = ref(false)
  let guesses = ref(Array())
  let currentGuess = ref(0)
  let playerGuessCount = ref(1)
  let givenWordCount = 0
  let gameResults = ref('')
  let finished = ref(false)
  let correct = ref(false)
  let notInDictionary = ref(false)
  let usedHint = ref(false)
  let usedHintBefore = ref(0)
  let gaveUp = ref(false)
  let currentPosition = ref(0)

  for ( let key in params ) {
    let initialGuess = []
    if ( key.startsWith('s') ) {
      givenWordCount++
      for ( let i=0; i < wordLength; i++ ) {
        initialGuess.push({ 'letter': params[key].charAt(i), 'state': 0, 'initialized': true, 'colored': true })
      }
      guesses.value.push(initialGuess)
    }
  }

  const addEmptyRow = function() {
    let initialGuess = []
    if ( revealStartingLetter ) {
      initialGuess.push({ 'letter': word.charAt(0), 'state': 4, 'initialized': true, 'colored': true, 'completed': false })
    } else {
      initialGuess.push({ 'letter': '', 'state': 0, 'initialized': false, 'colored': false, 'completed': false })
    }
    for ( let i=1; i < wordLength; i++ ) {
      initialGuess.push({ 'letter': '', 'state': 0, 'initialized': false, 'colored': false, 'completed': false })
    }
    guesses.value.push(initialGuess)
  }

  for ( let i=0; i < numberOfGuesses; i++ ){
    addEmptyRow()
  }

  if ( numberOfGuesses == 0 && wordLength != 0 ) {
    addEmptyRow()
  }

  let keyboardRows = ref([
    [
      {
        'letter': 'q',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'w',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'e',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'r',
        'state': 1,
        'colored': true
      },
      {
        'letter': 't',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'y',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'u',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'i',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'o',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'p',
        'state': 1,
        'colored': true
      }
    ],
    [
      {
        'letter': 'a',
        'state': 1,
        'colored': true
      },
      {
        'letter': 's',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'd',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'f',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'g',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'h',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'j',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'k',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'l',
        'state': 1,
        'colored': true
      },
      {
        'letter': '_',
        'state': 1,
        'colored': true
      }
    ],
    [
      {
        'letter': 'enter',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'z',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'x',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'c',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'v',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'b',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'n',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'm',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'del',
        'state': 1,
        'colored': true
      }
    ]
  ])

  const onKeyup = (e) => onKey(e.key)
  const tileClick = function(e) {
    if ( finished.value ){
      return
    }
    currentPosition.value = e.detail
  }
  window.addEventListener('keyup', onKeyup)
  const tileClickEvent = new Event('tile-click');
  window.addEventListener('tile-click',tileClick)
  onUnmounted(() => {
    window.removeEventListener('keyup', onKeyup)
    window.removeEventListener('tile-click',tileClick)
  })

  onMounted(() => {
    var tooltipTriggerList = [].slice.call(document.querySelectorAll('[data-bs-toggle="tooltip"]'))
    var tooltipList = tooltipTriggerList.map(
      function (tooltipTriggerEl) {
        return new bootstrap.Tooltip(tooltipTriggerEl)
    })
  })

  const onKey = function(key) {
    if (showModal.value) {
      return
    }
    if (/^[a-zA-Z_\-]$/.test(key)) {
      if (key == '-') {
        key = '_'
      }
      fillTile(key.toLowerCase())
    } else if (key === 'Backspace' || key === 'Delete') {
      clearTile()
    } else if (key === 'Enter') {
      completeRow(false)
    } else if ( key == 'ArrowLeft' && currentPosition.value > 0 ) {
      currentPosition.value--
    } else if ( ( key == 'ArrowRight' || key == ' ' ) && currentPosition.value < wordLength - 1 ) {
      currentPosition.value++
    }
  }

  const formInvalid = computed(() => {
    return ( newWordInvalid.value || newNumberOfGuessesInvalid.value || newStartingWordsInvalid.value.includes(true) )
  })

  const guessNotInDictionary = computed(() => {
    if ( guesses.value.length == 0 || currentGuess.value == guesses.value.length || currentGuess.value < 0 ){
      return false
    }

    let playerAnswer = guesses.value[currentGuess.value].map((e) => e['letter']).join('')
    if ( word == playerAnswer ){
      return false
    }

    if ( playerAnswer.length != wordLength || playerAnswer.match(/_/) ) {
      return false;
    }

    return !allWords[wordLength].includes(playerAnswer.toUpperCase())
  })

  const fillTile = function(key){
    let tile = {}
    if ( currentPosition.value == wordLength ){
      tile = guesses.value[currentGuess.value][currentPosition.value - 1]
    } else {
      tile = guesses.value[currentGuess.value][currentPosition.value]
    }
    tile['letter'] = key
    if ( currentPosition.value < wordLength - 1  ){
      currentPosition.value++
    }
  }

  const clearTile = function() {
    if ( currentPosition.value > 0 && guesses.value[currentGuess.value][currentPosition.value]['letter'] == '' ){
      currentPosition.value--
    }
    let tile = guesses.value[currentGuess.value][currentPosition.value]
    tile['letter'] = ''
  }

  const completeRow = function(skipAnimation) {
    let skip = skipAnimation ? 0 : 1
    let guess = guesses.value[currentGuess.value]
    for ( let i = 0; i < guess.length; i++){
      if ( guess[i]['letter'] === '' ) {
        return
      }
    }

    let answerLetters = word.split('')
    let playerAnswer = guess.map((e) => e['letter']).join('')
    correct.value = ( playerAnswer === word )

    if ( !correct.value && !allPossibleWords.includes(playerAnswer.toUpperCase()) && !skipAnimation ){
      showWordMissingMessage()
      return
    }
    let changedLetters = []
    let keyboardUpdates = []
    for ( let i=0; i < guess.length; i++ ) {
      if ( guess[i]['letter'] === answerLetters[i] ) {
        answerLetters[i] = null
        changedLetters.push(i)
        keyboardUpdates.push([guess[i]['letter'],4])
        setTimeout( () => {
          guess[i]['state'] = 4
        }, 150 * (i) * skip)
        setTimeout( () => {
          guess[i]['colored'] = true
        }, (450 + 150 * (i)) * skip)
      }
    }

    for ( let i=0; i < guess.length; i++ ) {
      if ( !changedLetters.includes(i) && answerLetters.includes(guess[i]['letter']) ){
        answerLetters[answerLetters.indexOf(guess[i]['letter'])] = null
        changedLetters.push(i)
        keyboardUpdates.push([guess[i]['letter'],3])
        setTimeout( () => {
          guess[i]['state'] = 3
        }, 150 * (i) * skip)
        setTimeout( () => {
          guess[i]['colored'] = true
        }, (450 + 150 * (i)) * skip)
      }
    }

    for ( let i=0; i < guess.length; i++ ) {
      if ( !changedLetters.includes(i) ) {
        keyboardUpdates.push([guess[i]['letter'],2])
        setTimeout( () => {
          guess[i]['state'] = 2
        }, 150 * (i) * skip)
        setTimeout( () => {
          guess[i]['colored'] = true
        }, (450 + 150 * (i)) * skip)
      }
    }

    setTimeout( () => {
      for ( let i=0; i < keyboardUpdates.length; i++ ) {
        updateKeyboard(keyboardUpdates[i][0],keyboardUpdates[i][1])
      }
      if ( correct.value || finished.value ) {
        genGameResults()
        showWinModal.value = true
      }
    }, 300 * guess.length * skip)

    if ( correct.value ) {
      finished.value = true
      currentPosition.value = -1
      currentGuess.value = -1
      return
    }

    currentGuess.value++
    currentPosition.value = 0
    if (!skipAnimation ){
      if (playerGuessCount.value < numberOfGuesses || numberOfGuesses == 0 ){
        playerGuessCount.value++
      } else {
        finished.value = true
      }
      if ( numberOfGuesses == 0 ) {
        addEmptyRow()
      }
    }
  }

  let newWordNotInDictionary = computed(() => {
    if (newWord.value.length < 2 || newWord.value.length > 15) {
      return false
    }
    return ( !allWords[newWord.value.length].includes(newWord.value.toUpperCase()) )
  })

  const showWordMissingMessage = function() {
    notInDictionary.value = true
    setTimeout(() => {
      notInDictionary.value = false
    },1500)
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

  const genGameResults = function() {
    let emoji = ['','','⬜','🟨','🟩']
    let hintEmoji = ['','','⚪','🟡','🟢']
    let results = 'I ' + (correct.value ? 'solved ' : 'did not solve ') + ( creator ? creator + "'s" : 'this' ) + " Custom Wordle on the Wordlelator! " + (correct.value ? playerGuessCount.value : 'X' ) + '/' + (numberOfGuesses > 0 ? numberOfGuesses : '∞' ) + "\n"
    for ( let i=0; i < guesses.value.length; i++ ){
      let row = []
      for ( let x=0; x < wordLength; x++ ) {
        if ( guesses.value[i][x]['state'] == 0 ){
          break
        }
        if ( usedHint.value && usedHintBefore.value == i){
          row.push(hintEmoji[guesses.value[i][x]['state']])
        } else {
          row.push(emoji[guesses.value[i][x]['state']])
        }
      }
      if ( row.length == word.length ){
        results += row.join('')
        results += "\n"
      }
    }
    if (gaveUp.value) {
      for ( let x=0; x < wordLength; x++ ) {
        results += '❌'
      }
      results += "\n"
    }
    results += document.location
    gameResults.value = results
    if (!correct.value){
      msg.value = 'Too Bad!'
    }
  }

  for ( let i=0; i < givenWordCount; i++ ){
    completeRow(true)
  }

  let newWordInvalid = computed(() => {
    return !newWord.value.match(/^[a-zA-Z]+$/) || newWord.value.length < 2 || newWord.value.length > 15
  })

  let newNumberOfGuessesInvalid = computed(() => {
    return newNumberOfGuesses.value === '' || ( newNumberOfGuesses.value != '' && newNumberOfGuesses.value < 0 )
  })

  let newStartingWordsInvalid = computed(() => {
    let results = []
    for ( let i=0; i < newStartingWords.value.length; i++){
      results.push( (newStartingWords.value[i].length != newWord.value.length || !newStartingWords.value[i].match(/^[a-zA-Z]+$/)) && newStartingWords.value[i].length != 0 )
    }
    return results
  })

  const copy = function() {
    if ( newWordInvalid.value ){
      return
    }
    generateUrl()
    const text = "Try my custom Wordle on the Wordlelator!\n" + newUrl.value
    navigator.clipboard.writeText(text)
    let span = document.getElementById('copiedMessage')
    let classes = span.className
    span.className += 'shown'
    setTimeout(() => {
      span.className = classes
    }, 2000)
  }

  const copyUrl = function() {
    if ( newWordInvalid.value ){
      return
    }
    generateUrl()
    navigator.clipboard.writeText(newUrl.value)
    let span = document.getElementById('copiedJustMessage')
    let classes = span.className
    span.className += 'shown'
    setTimeout(() => {
      span.className = classes
    }, 2000)
  }

  const copyResults = function() {
    navigator.clipboard.writeText(gameResults.value)
    let span = document.getElementById('copiedResultsMessage')
    let classes = span.className
    span.className += 'shown'
    setTimeout(() => {
      span.className = classes
    }, 2000)
  }

  const generateUrl = function() {
    if ( newWordInvalid.value || newNumberOfGuessesInvalid.value || newStartingWordsInvalid.value.includes(true) ){
      return
    }

    let o = {}
    o['w'] = encrypt(newWord.value.toLowerCase(),true)
    o['g'] = newNumberOfGuesses.value

    if ( newCreator.value != '' ) {
      o['c'] = newCreator.value
    }

    if ( newHint1.value != '' ){
      o['h1'] = newHint1.value
    }

    if ( newHint2.value != '' ){
      o['h2'] = encrypt(newHint2.value,false)
    }

    let count = 1
    for ( let i=0; i < newStartingWords.value.length; i++ ) {
      if ( newStartingWords.value[i].length > 0 ) {
        o['s' + count++] = newStartingWords.value[i].toLowerCase()
      }
    }

    if ( newRevealStartingLetter.value ) {
      o['r'] = 1
    }

    if ( newMessage.value != '' ) {
      o['m'] = encrypt(newMessage.value,false)
    }

    newUrl.value = 'https://theasylm.github.io/wordlelator/?p=' + JSURL.stringify(o)
  }

  let addNewStartingWord = function() {
    newStartingWords.value.push('')
    setTimeout(() => {document.getElementById('s' + (newStartingWords.value.length - 1)).focus()},250)
  }

  let gotoUrl = function() {
    if ( newWordInvalid.value ){
      return
    }
    generateUrl()
    window.open(newUrl.value, '_blank');
  }

  if ( hint1 != '' ){
    window.setTimeout( () => {
      document.getElementById('board').style.height ='calc(100vh - 22.5rem)';}, 100)
  }

  let openHintModal = function() {
    showHintModal.value = false
    usedHint.value = true
    usedHintBefore.value = currentGuess.value
  }

  let giveUp = function() {
    showConfirmModal.value = false;
    finished.value = true
    gaveUp.value = true
    genGameResults()
    showWinModal.value = true
  }

  let clearForm = function() {
    newWord.value = ''
    newNumberOfGuesses.value = 6
    newCreator.value = ''
    newHint1.value = ''
    newHint2.value = ''
    newMessage.value = ''
    newRevealStartingLetter.value = false
    newUrl.value = ''
    newStartingWords.value = Array()
    document.getElementById('word').focus()
  }
</script>

<template>
  <div class="container">
    <div class="header row">
      <div class="col-md-4">
        <button @click="showFormModal" class="new-button btn btn-primary">
          <PencilIcon></PencilIcon>New Wordle
        </button>
      </div>
      <div class="col-md-4">
        <span class="title">Wordlelator</span>
      </div>
      <div class="col-md-4 help">
        <XIcon class="give-up-icon" @click="showConfirmModal = (true && !finished)"></XIcon>
        <ChartBarIcon :class="{ inactive: !finished }" @click="showWinModal = (true && finished)"></ChartBarIcon>
        <QuestionMarkCircleIcon @click="showHelpModal = true"></QuestionMarkCircleIcon>
        <LightBulbIcon :class="{ inactive: hint2 == '' }" @click="showHintModal = (true && hint2 != '')"></LightBulbIcon>
      </div>
    </div>
    <div class="info">
      <h5 class="warning-message" :class="{'shown': notInDictionary}">Word not in dictionary.</h5>
      <h3 v-if="hint1 != ''">{{hint1}}</h3>
      <div>
        <span class="creator" v-if="creator != ''">Creator: {{creator}}</span>
        <span class="guess-counter" v-if="wordLength > 0">L: {{wordLength}}</span>
        <span class="guess-counter" v-if="wordLength > 0">Guess: {{playerGuessCount}}/{{numberOfGuesses > 0 ? numberOfGuesses : '∞'}}</span>
      </div>
    </div>
    <Board :guesses="guesses" :guessNotInDictionary="guessNotInDictionary" :currentGuess="currentGuess" :currentPosition="currentPosition" :wordLength="wordLength"></Board>
    <Keyboard :rows="keyboardRows"></Keyboard>
    <div class="footer">
      Custom Wordle creation tool by theasylm.
    </div>
    <div>
      <vue-final-modal
        name="newWordle"
        classes="modal-container newModal"
        :click-to-close="false"
        :esc-to-close="true"
        v-model="showModal"
        content-class="modal-content"
        :max-width="500"
      >
        <div class="formIcons">
          <div class="reset-modal">
            <RefreshIcon @click="clearForm"></RefreshIcon>
          </div>
          <div class="close-modal-div">
           <XIcon @click="showModal = false"></XIcon>
         </div>
        </div>
        <span class="modal__title">New Custom Wordle</span>
        <div class="modal__content">
          <div class="mb-3 row">
            <div class="col-sm-4 col-form-label">
              <label for="word">Word</label>
              <div data-bs-toggle="tooltip" title="Secret word to be guessed.<br/>2-15 Letters. A-Z only.<br/>Does not need to be in dictionary." data-bs-placement="auto" data-bs-trigger="click" data-bs-html="true" class="tooltip-icon" >
                <QuestionMarkCircleIcon ></QuestionMarkCircleIcon>
              </div>
            </div>
            <div class="col-sm-8">
              <input type="text" class="form-control" id="word" v-model="newWord" :class="{'has-error': newWordInvalid }"/>
              <span class="new-word-warning-message" :class="{'shown': newWordNotInDictionary }">Warning: word not in dictionary.</span>
            </div>
          </div>
          <div class="mb-3 row">
            <div class="col-sm-4 col-form-label">
              <label for="guesses">Number of Guesses</label>
              <div data-bs-toggle="tooltip" title="Does not count starting words.<br/>Set to 0 for infinite." data-bs-placement="auto" data-bs-trigger="click" data-bs-html="true" class="tooltip-icon" >
                <QuestionMarkCircleIcon ></QuestionMarkCircleIcon>
              </div>
            </div>
            <div class="col-sm-8">
              <input type="number" class="form-control" id="guesses" v-model="newNumberOfGuesses"  :class="{'has-error': newNumberOfGuessesInvalid }"/>
            </div>
          </div>
          <div class="mb-3 row">
            <div class="col-sm-4 col-form-label">
              <label for="startingLetter">Reveal Starting Letter</label>
              <div data-bs-toggle="tooltip" title="Automatically places starting letter." data-bs-placement="auto" data-bs-trigger="click" data-bs-html="true" class="tooltip-icon" >
                <QuestionMarkCircleIcon ></QuestionMarkCircleIcon>
              </div>
            </div>
            <div class="col-sm-8 slider">
              <div class="form-check form-switch">
                <input class="form-check-input" type="checkbox" role="switch" id="startingLetter" v-model="newRevealStartingLetter">
              </div>
            </div>
          </div>
          <div class="mb-3 row">
            <div class="col-sm-4 col-form-label">
              <label for="creator">Creator</label>
              <div data-bs-toggle="tooltip" title="Your name.<br/>Optional." data-bs-placement="auto" data-bs-trigger="click" data-bs-html="true" class="tooltip-icon" >
                <QuestionMarkCircleIcon ></QuestionMarkCircleIcon>
              </div>
            </div>
            <div class="col-sm-8">
              <input type="text" class="form-control" id="creator" v-model="newCreator"/>
            </div>
          </div>
          <div class="mb-3 row">
            <div class="col-sm-4 col-form-label">
              <label for="hint1">Title</label>
              <div data-bs-toggle="tooltip" title="Title to be displayed above puzzle.<br/>Optional." data-bs-placement="auto" data-bs-trigger="click" data-bs-html="true" class="tooltip-icon" >
                <QuestionMarkCircleIcon ></QuestionMarkCircleIcon>
              </div>
            </div>
            <div class="col-sm-8">
              <input type="text" class="form-control" id="hint1" v-model="newHint1"/>
            </div>
          </div>
          <div class="mb-3 row">
            <div class="col-sm-4 col-form-label">
              <label for="hint2">Hidden Hint</label>
              <div data-bs-toggle="tooltip" title="Hint initially hidden from player.<br/>Optional." data-bs-placement="auto" data-bs-trigger="click" data-bs-html="true" class="tooltip-icon" >
                <QuestionMarkCircleIcon ></QuestionMarkCircleIcon>
              </div>
            </div>
            <div class="col-sm-8">
              <input type="text" class="form-control" id="hint2" v-model="newHint2"/>
            </div>
          </div>
          <div class="mb-3 row">
            <div class="col-sm-4 col-form-label">
              <label for="message">Completion Message</label>
              <div data-bs-toggle="tooltip" title="Message to be displayed upon success.<br/>Optional." data-bs-placement="auto" data-bs-trigger="click" data-bs-html="true" class="tooltip-icon" >
                <QuestionMarkCircleIcon ></QuestionMarkCircleIcon>
              </div>
            </div>
            <div class="col-sm-8">
              <input type="text" class="form-control" id="message" v-model="newMessage"/>
            </div>
          </div>
          <div class="mb-3 row" v-for="(sw,index) in newStartingWords">
            <div class="col-sm-4 col-form-label">
              <label for="message">Starting Word</label>
            </div>
            <div class="col-sm-8">
              <input type="text" class="form-control" :id="'s' + index" v-model="newStartingWords[index]" :class="{'has-error': newStartingWordsInvalid[index]}" />
            </div>
          </div>
          <div class="mb-3 row">
            <div class="col-sm-4">
            </div>
            <div class="col-sm-8 left">
              <button class="btn btn-primary" @click="addNewStartingWord">Add Starting Word</button>
              <div data-bs-toggle="tooltip" title="Word to be automatically applied.<br/>No limit.<br/>Does not need to be in dictionary.<br/>Optional." data-bs-placement="auto" data-bs-trigger="click" data-bs-html="true" class="tooltip-icon" >
                <QuestionMarkCircleIcon ></QuestionMarkCircleIcon>
              </div>
            </div>
          </div>
        </div>
        <div class="modal__action">
          <div class="mb-3 row">
            <div class="col-4">
              <button @click="gotoUrl" class="btn btn-primary" :disabled="formInvalid">Go to Puzzle</button>
            </div>
            <div class="col-4">
              <button @click="copy" class="btn btn-primary" :disabled="formInvalid">Share Link</button><br/>
              <span id="copiedMessage">Copied!</span>
            </div>
            <div class="col-4">
              <button @click="copyUrl" class="btn btn-primary" :disabled="formInvalid">Copy Link</button><br/>
              <span id="copiedJustMessage">Copied!</span>
            </div>
          </div>
        </div>
      </vue-final-modal>
      <vue-final-modal
        name="hintModal"
        classes="modal-container"
        :click-to-close="false"
        :esc-to-close="true"
        v-model="showHintModal"
        content-class="modal-content"
        :max-width="500"
      >
        <div class="close-modal-div">
          <XIcon @click="openHintModal"></XIcon>
        </div>
        <div class="modal__content hint">
          <div class="hint-div">
            <div class="hint-span">{{hint2}}</div>
          </div>
        </div>
      </vue-final-modal>
      <vue-final-modal
        name="hintModal"
        classes="modal-container"
        :click-to-close="false"
        :esc-to-close="true"
        v-model="showConfirmModal"
        content-class="modal-content"
        :max-width="500"
      >
        <div class="close-modal-div">
          <XIcon @click="showConfirmModal = false"></XIcon>
        </div>
        <div class="modal__content confirm">
          <div class="hint-div">
            <div class="warning">Are you sure you wish to give up?</div>
          </div>
        </div>
        <div class="modal__action">
          <div class="mb-3 row">
            <div class="col-sm-4">
              <button @click="showConfirmModal = false" class="btn btn-primary">Keep Thinking</button>
            </div>
            <div class="col-sm-4">
              <button @click="giveUp" class="btn btn-danger">Give Up</button><br/>
              <span id="copiedMessage">Copied!</span>
            </div>
          </div>
        </div>
      </vue-final-modal>
      <vue-final-modal
        name="winModal"
        classes="modal-container"
        :click-to-close="false"
        :esc-to-close="true"
        v-model="showWinModal"
        content-class="modal-content"
        :max-width="500"
      >
        <div class="close-modal-div">
          <XIcon @click="showWinModal = false"></XIcon>
        </div>
        <span class="modal__title" id="win-msg">{{msg}}</span>
        <div class="modal__content">
          <div class="solution" v-if="finished && !correct">Solution: {{word.toUpperCase()}}</div>
          <button class="btn btn-primary share-button" @click="copyResults">Share results</button><br/>
          <span id="copiedResultsMessage">Copied!</span>
        </div>
      </vue-final-modal>
      <vue-final-modal
        name="helpModal"
        classes="modal-container help-modal"
        :click-to-close="false"
        :esc-to-close="true"
        v-model="showHelpModal"
        content-class="modal-content"
        :max-width="600"
      >
        <div class="close-modal-div">
          <XIcon @click="showHelpModal = false"></XIcon>
        </div>
        <div class="modal__content">
          <h2>How to Play</h2>
          <div class="mb-3 row">
            <div class="col-sm-12">
              Guess the Wordle in the given number of tries. After each guess, the tiles will be colored to indicate how close to the target word your guess was.
              <img src="./assets/green_clue.png"/>
              Green indicates the N is in the correct spot.
              <img src="./assets/yellow_clue.png"/>
              Yellow indicates the U is in the word, but in another position.
              <p><img src="./assets/grey_clue.png"/>
              Grey indicates the P is not in the word.</p>
              <p>Your creator may have left a hint for you. Look to see if the light bulb is "lit up". If it is, the creator has left a hint for you to use when you're ready.</p>
              <p>The given number of tries is set by the creator and can be no limit. Look above the grid to see how many guesses you have. </p>
              <p>Finally, if you wish to give up, you can hit the red X. You will be asked to confirm your decision.</p>
              <hr/>
            </div>
          </div>
          <h2>How to Create</h2>
          <div class="row">
            <div class="col-sm-12">
              <p>
              To create your own custom Wordle, hit the 'New Wordle' button. On the form presented, you can enter a number of different options. The only required entries are Word and Number of Guesses.</p>
              <table class="table">
                <tr>
                  <td class="col-sm-3">Word</td>
                  <td class="col-sm-9">
                    Set a word of length 2-15. Limited as the dictionary only handles lengths 2-15.<br/>
                    Words are not required to be in the dictionary to be the secret word, allowing for proper nouns and loan words.<br/>
                    Words can ONLY contain the letters A-Z. No digits or other special characters.
                  </td>
                </tr>
                <tr>
                  <td class="col-sm-3">Number of Guesses</td>
                  <td class="col-sm-9">
                    This is the number of guesses the player has to guess your word, after any starting words you give them.<br/>
                    You may set this value to 0 to allow for infinite guesses.
                  </td>
                </tr>
                <tr>
                  <td class="col-sm-3">Reveal Starting Letter</td>
                  <td class="col-sm-9">
                    Reveals the starting letter immediately to the solver and automatically places it.<br/>
                    Recommended for long words.
                  </td>
                </tr>
                <tr>
                  <td class="col-sm-3">Creator</td>
                  <td class="col-sm-9">Take pride in your puzzle by optionally adding your name.</td>
                </tr>
                <tr>
                  <td class="col-sm-3">Title</td>
                  <td class="col-sm-9">
                    A title that will be displayed above the puzzle.<br/>
                    Can be used to denote content-specific puzzles, for example.
                  </td>
                </tr>
                <tr>
                  <td class="col-sm-3">Hidden Hint</td>
                  <td class="col-sm-9">
                    This is a hint that is hidden from the player initially.<br/>
                    They can access it by pressing the light bulb icon.<br/>
                    Good for definitions.
                  </td>
                </tr>
                <tr>
                  <td class="col-sm-3">Completion Message</td>
                  <td class="col-sm-9">Custom message to be displayed to a successful solver of your puzzle.</td>
                </tr>
                <tr>
                  <td class="col-sm-3">Starting Words</td>
                  <td class="col-sm-9">
                    You can specify any number of starting words.<br/>
                    These will be automatically applied and clued.<br/>
                    Guess counts do NOT count the provided starting words as guesses.
                  </td>
                </tr>
              </table>
            </div>
            <div class="col-sm-12">
              Once you've filled in all the boxes you're interested in, hit either the 'Go to Puzzle' button (good for testing your puzzle); the 'Share Link' button, which will copy the puzzle link with a handy message ready for pasting in chat, email, or wherever; or the 'Copy Link' button, which will copy just the link to the puzzle.
            </div>
          </div>
        </div>
      </vue-final-modal>
    </div>
  </div>
</template>

<style>
  body, html {
    height: 100%;
  }
  body {
    overflow: hidden;
  }
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #efefef;
    /**background-color: #08111b;**/
    background-color: #011637;
    height: 100%;
    padding-top: 1em;
  }
  .header {
  }
  .title {
    font-size: 2em;
  }
  .info {
    position: relative;
  }
  .warning-message {
    visibility: hidden;
    margin: 0;
  }
  .warning-message.shown {
    visibility: visible;
  }
  .new-word-warning-message {
    visibility: hidden;
    color: #ffc107;
  }
  .new-word-warning-message.shown {
    visibility: visible;
  }
  #url {
    width:  100%;
  }
  #copiedMessage, #copiedResultsMessage, #copiedJustMessage {
    visibility: hidden;

  }
  #copiedMessage.shown, #copiedResultsMessage.shown, #copiedJustMessage.shown {
    visibility: visible;
    opacity: 0;
    animation: fade 2s linear forwards;
  }
  .share-button {
    margin-top: 2rem;
  }
  .close-modal-div, .reset-modal {
    position: absolute;
    cursor: pointer;
  }

  .close-modal-div {
    margin-top: -.5rem;
    width: 38px;
    right: .5rem;
  }

  .reset-modal {
    margin-top: -.45rem;
    width: 32px;
    right:  42px;
  }

  .left-align {
    text-align: left;
  }
  .help svg {
    width: 36px;
    margin: 0 .25rem;
    cursor: pointer;
  }
  .help svg.inactive {
    opacity: 50%;
    cursor:  default;
  }
  .help-modal img {
    width: 100%;
  }
  .help-modal h2 {
    text-align: center;
  }
  .help-modal hr {
    height: 2px;
    background-color: #efefef;
    border: none;
    opacity: 1;
    margin-bottom: 0;
  }
  .solution {
    font-size: 1.5rem;
    font-weight: 500;
    margin-top: 1.5rem;
  }
  @keyframes fade {
    0%,100% { opacity: 0 }
    50% { opacity: 1 }
  }
  .add-link {
    cursor:  pointer;
    color: #efefef;
    text-decoration: none;
  }
  .add-link:hover {
    color: #efefef;
  }

  .info span {
    margin: 0 .5rem;
    font-size:  1.25rem;
  }
  table.table {
    color: #efefef;
    border-top: 2px solid;
    border-bottom: 2px solid;
  }
  .has-error {
    border-color: #842029 !important;
    border-width: 4px;
  }
  .hint-span {
    font-size:  2.5rem;
  }
  .hint-div {
    min-height: 10rem;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  #win-msg {
    font-size:  2.25rem;
  }
  .give-up-icon {
    color:  #842029;
    width:  46px !important;
  }
  .warning {
    font-size: 2rem;

  }
  .slider {
    display: flex;
    align-items: center;
  }
  .tooltip-icon {
    margin-left: 0.5rem;
    margin-top: -.5rem;
    padding: 0;
    width: 1.25rem;
    display: inline-block;
  }
  .tooltip-icon svg {
    margin-top: -.25rem;
  }
  .bs-tooltip-end .tooltip-arrow::before, .bs-tooltip-bottom .tooltip-arrow::before , .bs-tooltip-top .tooltip-arrow::before, .bs-tooltip-start .tooltip-arrow::before {
    border-left-color: transparent;
    border-top-color:  transparent;
    border-bottom-color:  transparent;
    border-right-color: transparent;
  }
  .tooltip-inner {
    background-color: #011637;
    border: 1px solid white;
    max-width: 300px;
  }
  .tooltip.show {
    opacity: 1;
  }
  .left {
    justify-content: left;
    display: flex;
  }
  .left .tooltip-icon {
    margin-top: 0.5rem;
  }
  .footer {
    position: absolute;
    bottom: 0.5rem;
    font-size: smaller;
    width: 100%;
    left: 0;
  }
</style>
<style scoped>
  ::v-deep .modal-content {
    position: relative;
    display: flex;
    flex-direction: column;
    max-height: 90%;
    margin: 0 1rem;
    padding: 1rem;
    border: 1px solid #efefef;
    border-radius: 0.25rem;
    background: #011637;
    width: 700px;
    min-height: 10rem;
  }
  .modal__content {
    scrollbar-color: white;
  }
  .modal__content::-webkit-scrollbar {
    scrollbar-color: white;
  }
  ::v-deep .help-modal .modal-content {
    width:  650px;
    text-align: left;
  }
  .help-modal ul {
    list-style:  none;
  }
  .modal__title {
    font-size: 1.5rem;
    font-weight: 700;
  }
  ::v-deep .modal-container {
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .modal__content {
    flex-grow: 1;
    overflow-y: auto;
  }
  .modal__close {
    position: absolute;
    top: 0.5rem;
    right: 1.5rem;
  }
  .modal__action {
    padding: 1rem 0 0;
  }
  .new-button {
    width: 10rem;
  }
  .new-button svg {
    float: right;
    width: 24px;
  }
  @media (max-width: 584px){
    .newModal .btn {
      margin: .5rem 0;
      height: 4rem;
    }
    .left .tooltip-icon {
      margin-top: 1.75rem;
    }
    .left button {
      width: 90%;
    }

  }
</style>
