<script setup>
import { ref, watch, reactive } from 'vue'
import words from './words'
import plot from './components/plot.vue'

function randomWord() {
  return words[Math.floor(Math.random() * words.length)]
}

const word = ref("")
const userInput = ref('')
const wordTime = ref(0);
const timePerLetter = ref(0);
const started = ref(false);
const done = ref(false);
const inputElement = ref(null);
const history = ref(JSON.parse(localStorage.getItem('history') || '[]'));
const showingIndex = ref(false)
const commandMode = ref(false)
const command = reactive({ command: '', params: '', exists: false })
const downloaderElem = ref(null)

const commands = {
  'toggleIndex': function () {
    showingIndex.value = !showingIndex.value
  },
  'downloadHistory': function () {
    const blob = new Blob([JSON.stringify(history.value)], { type: 'application/json' })
    const objectURL = URL.createObjectURL(blob)
    downloaderElem.value.setAttribute("href", objectURL);
    downloaderElem.value.setAttribute("download", "typing test log.json");
    downloaderElem.value.click();
  },
  'loadHistory': function (params) {
    history.value = JSON.parse(params)
  },
  'clearHistory': function () {
    history.value = []
  },
  'deleteIndex': function (params) {
    if (parseInt(params) < 0) {
      params = history.value.length + parseInt(params)
    }
    history.value = history.value.filter((item, index) => index !== parseInt(params))
  },
  'next': function (params) {
    next(params)
  }
}

watch(history, (newValue) => {
  localStorage.setItem('history', JSON.stringify(newValue));
});

let startTime = 0;

watch(userInput, (value) => {
  if (value.length > 0 && value[0] == "!" && !started.value) {
    commandMode.value = true;
    command.command = value.slice(1, value.indexOf(' ') > -1 ? value.indexOf(' ') : value.length);
    command.params = value.slice(value.indexOf(' ') > -1 ? value.indexOf(' ') + 1 : value.length);
    command.exists = commands[command.command] != undefined;
  } else {
    commandMode.value = false;
    if (!started.value && value.length > 0) {
      started.value = true;
      startTime = performance.now();
    }
    if (value === word.value) {
      wordTime.value = performance.now() - startTime;
      timePerLetter.value = wordTime.value / word.value.length;
      started.value = false;
      done.value = true;
      history.value = [...history.value, {
        word: word.value,
        time: Math.round(wordTime.value),
        timePerLetter: Math.round(timePerLetter.value)
      }]
    }
  }
})

function stop() {
  started.value = false;
  userInput.value = '';
  done.value = false;
  setTimeout(() => {
    inputElement.value && inputElement.value.focus();
  }, 1);
}

function next(nextWord = null) {
  if (nextWord) {
    word.value = nextWord;
  } else {
    word.value = randomWord();
  }
  userInput.value = '';
  done.value = false;
  setTimeout(() => {
    inputElement.value && inputElement.value.focus();
  }, 1);
}

function commandGo() {
  if (command.exists) {
    commands[command.command](command.params);
  }
}

window.addEventListener('keydown', (e) => {
  if (e.key == "Escape") {
    stop();
  }
  if (e.key == "Enter") {
    if (commandMode.value) {
      commandGo();
    }
    if (done.value) {
      next();
    }
  }
})

next()
</script>

<template>
  <div class="test">
    <h1>{{ word }}</h1>
    <input type="text" v-model="userInput" :class="{ typing: started, done: done, commandMode: commandMode }"
      class="textInput" style="outline: none;" autocomplete="off" :disabled="done" tabindex="0" ref="inputElement"
      autofocus>
    <p style="font-size: 1.2rem;" v-if="done">
      {{ Math.round(wordTime) }}ms {{ Math.round(timePerLetter) }}ms/letter
    </p>
    <div class="buttons">
      <button class="stopButton" v-if="started" @click="stop()">stop (esc)</button>
      <button class="againButton" v-if="done" @click="stop()">again (esc)</button>
      <button class="nextButton" v-if="done" @click="next()">next (enter)</button>
      <button class="goButton" v-if="commandMode && command.exists" @click="commandGo()">go (enter)</button>
    </div>
  </div>

  <div class="plotArea">
    <plot :history="history" @word-click="word => next(word)" :showIndex="showingIndex" />
  </div>

  <h1 class="pageTitle">
    1word
  </h1>

  <a style="display: none;" ref="downloaderElem"></a>
</template>

<style>
@import './global.css';

@keyframes blinkRedBorder {
  from {
    border-color: #f00;
  }

  to {
    border-color: #222;
  }
}

.textInput {
  background-color: transparent;
  color: white;
  font-size: 24px;
  padding: 6px;
  border-radius: 6px;
  border: 2px solid #222;
  transition: 100ms;
}

.textInput:focus {
  border-color: #444;
}

.textInput.typing {
  border-color: #f00;
  animation: blinkRedBorder 25ms linear infinite alternate;
}

.textInput.done {
  border-color: #0f0;
}

.textInput.commandMode {
  border-color: #00f;
}

.buttons {
  display: flex;
  align-items: center;
  gap: 8px;
}

button {
  color: black;
  border: none;
  padding: 3px 6px;
  border-radius: 4px;
  font-size: inherit;
  font-weight: bold;
}

.stopButton {
  background-color: red;
}

.nextButton {
  background-color: #0F0;
}

.againButton {
  background-color: #FC0;
}

.goButton {
  background-color: #00F;
}

.test {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
  position: static;
  z-index: 20;
}

.test>* {
  margin: 0;
}

.plotArea {
  position: absolute;
  inset: 0;
  overflow: hidden;
  z-index: 10;
}

.pageTitle {
  position: absolute;
  top: 16px;
  right: 16px;
  font-weight: 200;
  font-size: 48px;
  color: #FFF3;
  margin: 0;
}
</style>
