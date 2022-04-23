<script setup>
  import { $vfm, VueFinalModal, ModalsContainer } from 'vue-final-modal'
  import { ref, onUnmounted, computed } from 'vue'
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
      initialGuess.push({ 'letter': word.charAt(0), 'state': 4, 'initialized': true, 'colored': true })
    } else {
      initialGuess.push({ 'letter': '', 'state': 0, 'initialized': false, 'colored': false })
    }
    for ( let i=1; i < wordLength; i++ ) {
      initialGuess.push({ 'letter': '', 'state': 0, 'initialized': false, 'colored': false })
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
  window.addEventListener('keyup', onKeyup)
  onUnmounted(() => {
    window.removeEventListener('keyup', onKeyup)
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
      return
    }

    currentGuess.value++
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
    return newNumberOfGuesses.value < 0
  })

  let newStartingWordsInvalid = computed(() => {
    let results = []
    for ( let i=0; i < newStartingWords.value.length; i++){
      results.push( (newStartingWords.value[i].length != newWord.value.length || !newStartingWords.value[i].match(/^[a-zA-Z]+$/)) && newStartingWords.value[i].length != 0 )
    }
    return results
  })

  const copy = function() {
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
  }
</script>

<template>
  <div class="container">
    <div class="header row">
      <div class="col-md-3">
        <button @click="showFormModal" class="new-button btn btn-primary">
          <PencilIcon></PencilIcon>New Wordle
        </button>
      </div>
      <div class="col-md-6">
        <span class="title">Wordlelator</span>
      </div>
      <div class="col-md-3 help">
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
        <span class="guess-counter" v-if="wordLength > 0">Guess: {{playerGuessCount}}/{{numberOfGuesses > 0 ? numberOfGuesses : '∞'}}</span>
      </div>
    </div>
    <Board :guesses="guesses"></Board>
    <Keyboard :rows="keyboardRows"></Keyboard>
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
            <label for="word" class="col-sm-4 col-form-label" >Word</label>
            <div class="col-sm-8">
              <input type="text" class="form-control" id="word" v-model="newWord" :class="{'has-error': newWordInvalid }"/>
              <span class="new-word-warning-message" :class="{'shown': newWordNotInDictionary }">Warning: word not in dictionary.</span>
            </div>
          </div>
          <div class="mb-3 row">
            <label for="guesses" class="col-sm-4 col-form-label">Number of Guesses</label>
            <div class="col-sm-8">
              <input type="number" class="form-control" id="guesses" v-model="newNumberOfGuesses"  :class="{'has-error': newNumberOfGuessesInvalid }"/>
            </div>
          </div>
          <div class="mb-3 row">
            <label for="startingLetter" class="col-sm-4 col-form-label">Reveal starting letter</label>
            <div class="col-sm-8">
              <div class="form-check form-switch">
                <input class="form-check-input" type="checkbox" role="switch" id="startingLetter" v-model="newRevealStartingLetter">
              </div>
            </div>
          </div>
          <div class="mb-3 row">
            <label for="creator" class="col-sm-4 col-form-label">Creator</label>
            <div class="col-sm-8">
              <input type="text" class="form-control" id="creator" v-model="newCreator"/>
            </div>
          </div>
          <div class="mb-3 row">
            <label for="hint1" class="col-sm-4 col-form-label">Title</label>
            <div class="col-sm-8">
              <input type="text" class="form-control" id="hint1" v-model="newHint1"/>
            </div>
          </div>
          <div class="mb-3 row">
            <label for="hint2" class="col-sm-4 col-form-label">Hidden Hint</label>
            <div class="col-sm-8">
              <input type="text" class="form-control" id="hint2" v-model="newHint2"/>
            </div>
          </div>
          <div class="mb-3 row">
            <label for="message" class="col-sm-4 col-form-label">Completion Message</label>
            <div class="col-sm-8">
              <input type="text" class="form-control" id="message" v-model="newMessage"/>
            </div>
          </div>
          <div class="mb-3 row" v-for="(sw,index) in newStartingWords">
            <label :for="'s' + index" class="col-sm-4 col-form-label">Starting Word</label>
            <div class="col-sm-8">
              <input type="text" class="form-control" :id="'s' + index" v-model="newStartingWords[index]" :class="{'has-error': newStartingWordsInvalid[index]}" />
            </div>
          </div>
          <div class="mb-3 row">
            <div class="col-sm-12">
              <button class="btn btn-primary" @click="addNewStartingWord">Add Starting Word</button>
            </div>
          </div>
        </div>
        <div class="modal__action">
          <div class="mb-3 row">
            <div class="col-sm-4">
              <button @click="gotoUrl" class="btn btn-primary">Go to Puzzle</button>
            </div>
            <div class="col-sm-4">
              <button @click="copy" class="btn btn-primary">Share URL</button><br/>
              <span id="copiedMessage">Copied!</span>
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
                    Set a word of length 2-15 - limited as the dictionary only handles lengths 2-15.<br/>
                    Words are not required to be in the dictionary to be the secret word, allowing for proper nouns and loan words.
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
              Once you've filled in all the boxes you're interested in, hit either the 'Go to Puzzle' button (good for testing your puzzle), or the 'Share URL' button, which will copy the puzzle URL with a handy message ready for pasting in chat, email, or wherever.
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
  #copiedMessage, #copiedResultsMessage {
    visibility: hidden;

  }
  #copiedMessage.shown, #copiedResultsMessage.shown {
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
    height: 10rem;
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
    width: 600px;
    min-height: 10rem;
  }
  .modal__content {
    -ms-overflow-style: none;  /* IE and Edge */
    scrollbar-width: none;  /* Firefox */
  }
  .modal__content::-webkit-scrollbar {
    display: none;
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
  @media (max-width: 786px){
    .newModal .btn {
      margin: .5rem 0;
    }
  }
</style>
