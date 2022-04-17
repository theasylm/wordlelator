<script setup>
  import { $vfm, VueFinalModal, ModalsContainer } from 'vue-final-modal'
  import { ref, onUnmounted, computed } from 'vue'
  import Board from './components/Board.vue'
  import Keyboard from './components/Keyboard.vue'
  import CryptoJS from 'crypto-js'
  import JSURL from 'jsurl'
  import { PencilIcon, QuestionMarkCircleIcon, LightBulbIcon, XIcon, ChartBarIcon } from '@heroicons/vue/outline'
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
  const word = decrypt(params['w'],true)
  const wordLength = word.length > 0 ? word.length : 5
  const dictionary = './lib/js/' + wordLength + '.js'
  let allWords = undefined
  async function load() {
    let wordList = await import(dictionary);
    allWords = wordList['allWordsLength' + wordLength]
  }
  load()
  const numberOfGuesses = parseInt(params['g']) || 6
  const creator = params['c'] || ''
  const hint1 = params['h1'] || ''
  let hint2 = params['h2'] || ''
  if ( hint2 != '' ) {
    hint2 = decrypt(hint2,false)
  }
  let msg = ref('')
  msg.value = params['m'] || ''
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
  let newUrl = ref('')
  let showModal = ref(false)
  let showHintModal = ref(false)
  let showWinModal = ref(false)
  let showHelpModal = ref(false)
  let guesses = ref(Array())
  let currentGuess = ref(0)
  let playerGuessCount = ref(1)
  let givenWordCount = 0
  let gameResults = ref('')
  let finished = ref(false)
  let correct = ref(false)
  let notInDictionary = ref(false)

  for ( let key in params ) {
    let initialGuess = []
    if ( key.startsWith('s') ) {
      givenWordCount++
      for ( let i=0; i < wordLength; i++ ) {
        initialGuess.push({ 'letter': params[key].charAt(i), 'state': 0, 'initialized': true })
      }
      guesses.value.push(initialGuess)
    }
  }

  for ( let i=0; i < numberOfGuesses; i++ ){
    let initialGuess = []
    for ( let i=0; i < wordLength; i++ ) {
      initialGuess.push({ 'letter': '', 'state': 0, 'initialized': false })
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
    if (showModal.value) {
      return
    }
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
    let playerAnswer = guess.map((e) => e['letter']).join('')
    correct.value = ( playerAnswer === word )

    if ( !allWords.includes(playerAnswer.toUpperCase()) ){
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
      if ( correct.value || finished.value ) {
        genGameResults()
        showWinModal.value = true
      }
    }, 600 * guess.length * skip)

    if ( correct.value ) {
      finished.value = true
      return
    }

    currentGuess.value++
    if (!skipAnimation ){
      if (playerGuessCount.value < numberOfGuesses ){
        playerGuessCount.value++
      } else {
        finished.value = true
      }
    }
  }

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
    let emoji = ['','',':white_large_square:',':yellow_square:',':green_square:']
    let results = 'I ' + (correct.value ? 'solved ' : 'did not solve ') + ( creator ? creator + "'s" : 'this' ) + " Custom Wordle on the Wordlelator! " + (correct.value ? playerGuessCount.value : 'X' ) + '/' + numberOfGuesses + "\n"
    for ( let i=0; i < guesses.value.length; i++ ){
      for ( let x=0; x < wordLength; x++ ) {
        let result = emoji[guesses.value[i][x]['state']]
        if ( result == '' ){
          break;
        }
        results += emoji[guesses.value[i][x]['state']]
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
    return !newWord.value.match(/^[a-zA-Z]+$/)
  })

  let newNumberOfGuessesInvalid = computed(() => {
    return newNumberOfGuesses.value < 0
  })

  let newStartingWordsInvalid = computed(() => {
    let results = []
    for ( let i=0; i < newStartingWords.value.length; i++){
      results.push( newStartingWords.value[i].length != newWord.value.length )
    }
    return results
  })

  const copy = function() {
    navigator.clipboard.writeText(newUrl.value)
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
      o['s' + count++] = newStartingWords.value[i].toLowerCase()
    }

    if ( newMessage.value != '' ) {
      o['m'] = encrypt(newMessage.value,false)
    }

    newUrl.value = 'http://localhost:3000/?p=' + JSURL.stringify(o)
  }

  let addNewStartingWord = function() {
    newStartingWords.value.push('')
  }

  let gotoUrl = function() {
    if ( newUrl.value == '' ) {
      return
    }
    document.location = newUrl.value
  }
</script>

<template>
  <div class="container">
    <div class="header row">
      <div class="col-md-3">
        <button @click="showModal = true" class="new-button btn btn-primary">
          <PencilIcon></PencilIcon>New Wordle
        </button>
      </div>
      <div class="col-md-6">
        <span class="title">Wordlelator</span>
      </div>
      <div class="col-md-3 help">
        <ChartBarIcon :class="{ inactive: !finished }" @click="showWinModal = (true && finished)"></ChartBarIcon>
        <QuestionMarkCircleIcon @click="showHelpModal = true"></QuestionMarkCircleIcon>
        <LightBulbIcon :class="{ inactive: hint2 == '' }" @click="showHintModal = (true && hint2 != '')"></LightBulbIcon>
      </div>
    </div>
    <div class="info">
      <span class="warning-message" :class="{'shown': notInDictionary}">Word not in dictionary.</span>
      <span class="creator" v-if="creator != ''">Creator: {{creator}}</span>
      <span class="hint1" v-if="hint1 != ''">Hint: {{hint1}}</span>
      <span class="guess-counter">Guess: {{playerGuessCount}}/{{numberOfGuesses}}</span>
    </div>
    <Board :guesses="guesses"></Board>
    <Keyboard :rows="keyboardRows"></Keyboard>
    <div>
      <vue-final-modal
        name="newWordle"
        classes="modal-container newModal"
        :click-to-close="true"
        :esc-to-close="true"
        v-model="showModal"
        content-class="modal-content"
        :max-width="500"
      >
        <div class="close-modal-div">
          <XIcon @click="showModal = false"></XIcon>
        </div>
        <span class="modal__title">New Custom Wordle</span>
        <div class="modal__content">
          <div class="mb-3 row">
            <label for="word" class="col-sm-4 col-form-label" >Word</label>
            <div class="col-sm-8">
              <input type="text" class="form-control" id="word" v-model="newWord" :class="{'has-error': newWordInvalid }"/>
            </div>
          </div>
          <div class="mb-3 row">
            <label for="creator" class="col-sm-4 col-form-label">Creator</label>
            <div class="col-sm-8">
              <input type="text" class="form-control" id="creator" v-model="newCreator"/>
            </div>
          </div>
          <div class="mb-3 row">
            <label for="guesses" class="col-sm-4 col-form-label">Number of Guesses</label>
            <div class="col-sm-8">
              <input type="number" class="form-control" id="guesses" v-model="newNumberOfGuesses"  :class="{'has-error': newNumberOfGuessesInvalid }"/>
            </div>
          </div>
          <div class="mb-3 row">
            <label for="hint1" class="col-sm-4 col-form-label">Short, Visible Hint</label>
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
              <a class="add-link" @click="addNewStartingWord">Add Starting Word</a>
            </div>
          </div>
        </div>
        <div class="modal__action">
          <div class="mb-3 row">
            <div class="col-sm-12">
              <textarea id="url" onclick="this.focus();this.select()" readonly="readonly" v-model="newUrl"></textarea>
            </div>
          </div>
          <div class="mb-3 row">
            <div class="col-sm-4">
              <button @click="gotoUrl" class="btn btn-primary">Go to Puzzle</button>
            </div>
            <div class="col-sm-4">
              <button @click="copy" class="btn btn-primary">Copy URL</button><br/>
              <span id="copiedMessage">Copied!</span>
            </div>
            <div class="col-sm-4">
              <button @click="generateUrl" class="btn btn-primary">Generate URL</button>
            </div>
          </div>
        </div>
      </vue-final-modal>
      <vue-final-modal
        name="hintModal"
        classes="modal-container"
        :click-to-close="true"
        :esc-to-close="true"
        v-model="showHintModal"
        content-class="modal-content"
        :max-width="500"
      >
        <div class="close-modal-div">
          <XIcon @click="showHintModal = false"></XIcon>
        </div>
        <span class="modal__title">Hint</span>
        <div class="modal__content">
          <div class="mb-3 row">
            <div class="col-sm-12">
              <span class="hint-span">{{hint2}}</span>
            </div>
          </div>
        </div>
      </vue-final-modal>
      <vue-final-modal
        name="winModal"
        classes="modal-container"
        :click-to-close="true"
        :esc-to-close="true"
        v-model="showWinModal"
        content-class="modal-content"
        :max-width="500"
      >
        <div class="close-modal-div">
          <XIcon @click="showWinModal = false"></XIcon>
        </div>
        <span class="modal__title">{{msg}}</span>
        <div class="modal__content">
          <button class="btn btn-primary share-button" @click="copyResults">Share results</button><br/>
          <span id="copiedResultsMessage">Copied!</span>
        </div>
      </vue-final-modal>
      <vue-final-modal
        name="helpModal"
        classes="modal-container help-modal"
        :click-to-close="true"
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
              <img src="./assets/grey_clue.png"/>
              Grey indicates the P is not in the word.
              <hr/>
            </div>
          </div>
          <h2>How to Create</h2>
          <div class="row">
            <div class="col-sm-12">
              To create your own custom Wordle, hit the 'New Wordle' button. On the form presented, you can enter a number of different options. The only required entries are Word and Number of Guesses.
              <table class="table">
                <tr>
                  <td class="col-sm-3">Word</td>
                  <td class="col-sm-9">
                    Set a word of any length, though the dictionary only handles lengths 2-15.<br/>
                    Words are not required to be in the dictionary to be the secret word, allowing for proper nouns and loan words.
                  </td>
                </tr>
                <tr>
                  <td class="col-sm-3">Creator</td>
                  <td class="col-sm-9">Take pride in your puzzle by optionally adding your name.</td>
                </tr>
                <tr>
                  <td class="col-sm-3">Number of Guesses</td>
                  <td class="col-sm-9">This is the number of guesses the player has to guess your word, after any starting words you give them.</td>
                </tr>
                <tr>
                  <td class="col-sm-3">Short, Visible Hint</td>
                  <td class="col-sm-9">
                    A short hint that will be displayed above the puzzle.<br/>
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
              Once you've filled in all the boxes you're interested in, hit the 'Generate URL' button to generate the URL that will take you to your puzzle. Also provided are buttons to automatically copy the URL, as well as visiting your puzzle, so you can check for errors.
            </div>
          </div>
        </div>
      </vue-final-modal>
    </div>
  </div>
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
  .header {
  }
  .title {
    font-size: 2em;
  }
  .info {
    position: relative;
  }
  .warning-message {
    position: absolute;
    top: -1.2rem;
    display: none;
  }
  .warning-message.shown {
    display: block;
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
  .close-modal-div {
    width: 36px;
    position: absolute;
    right: .5rem;
    margin-top: -.5rem;
    cursor: pointer;
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
  }
  .warning-message {

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

  .info {
    display: flex;
    justify-content: center;
  }

  .info span {
    margin: 0 .5rem;
  }
  table.table {
    color: #efefef;
    border-top: 2px solid;
    border-bottom: 2px solid;
  }
  .has-error {
    border-color: #842029 !important;
    border-width: 3px;
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
    width: 500px;
    min-height: 10rem;
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
