<script setup lang="ts">

import { ref, computed } from 'vue'
import { processWords, lettersMatching } from './wordle'

// SEO and page info
useHead({
  title: 'Search assistant to help find words for the Wordle game',
  meta: [
    {
      name: 'A project to help win the game Wordle by searching strings with JavaScript/TypeScript',
      content: 'A project to help win the game Wordle by searching strings with JavaScript/TypeScript',
    }
  ],
  link: [
    /*
    {
      rel: 'canonical',
      href: 'https://conradswebsite.com/projects/search-assistant-to-help-find-words-for-the-wordle-game'
    }*/
  ],
  bodyAttrs: {
    class: 'test'
  },
  // script: [{ innerHTML: 'console.log(\'Hello world\')' }]
})

useSeoMeta({
  title: 'Search assistant to help find words for the Wordle game',
  ogTitle: 'Search assistant to help find words for the Wordle game',
  description: 'A project to help win the game Wordle by searching strings with JavaScript/TypeScript',
  ogDescription: 'A project to help win the game Wordle by searching strings with JavaScript/TypeScript',
  ogImage: 'https://conradswebsite.com/9word_searcher.jpg',
  twitterCard: 'summary_large_image',
})

// main variables
const notLetter = /[^a-zA-Z]/;
let checkboxValue = ref(false);
const processedWords: Ref<string[]> = ref([]);
const maxWordLength = 5;
let userInput = ref('');
let userInputExcludeLetters = ref('');
let userInputInWordSomewhere = ref('');
let invalidInput = ref(false);
let viewInstructions = ref(false);
let isRotated = ref(false); // menu styling variable

let duplicateLettersErrorMessage: Ref<string> = ref('');
const outputSuccessMessage: Ref<string> = ref("");



// only allow letters in 3rd input field
watch(userInputExcludeLetters, (newValue: string) => {
  userInputExcludeLetters.value = newValue.replace(/[^a-zA-Z]/g, '');
});



const inputLength = computed(() => `${userInput.value.length}/${maxWordLength}`);
const secondInputLength = computed(() => `${userInputInWordSomewhere.value.length}/${maxWordLength}`);

const getOutputSuccessMessage = () => {
  outputSuccessMessage.value = "Submission successful! Scroll down to see the results!";
}

// ERROR CHECKING FUNCTION
const checkForDuplicateLetters = (whichInput: string) => {
  const inputLetters = whichInput.toLowerCase().split('');
  const secondInputLetters = userInputInWordSomewhere.value.toLowerCase().split('');
  const excludeLetters = userInputExcludeLetters.value.toLowerCase().split('');

  const secondAndThirdDuplicates = secondInputLetters.filter(letter => excludeLetters.includes(letter));
  const firstInputDuplicates = inputLetters.filter((letter, index) =>
    /^[a-zA-Z]$/.test(letter) && (excludeLetters.includes(letter) || secondInputLetters[index] === letter)
  );

  if (firstInputDuplicates.length > 0 || secondAndThirdDuplicates.length > 0) {
    generateErrorMessage(firstInputDuplicates, secondAndThirdDuplicates);
    invalidInput.value = true;
    return true;
  }
  return false;
};

const generateErrorMessage = (firstDuplicates: string[], secondDuplicates: string[]) => {
  let messageParts = [];
  if (firstDuplicates.length) {
    messageParts.push(`The error letter${firstDuplicates.length > 1 ? 's are' : ' is'} <b>${firstDuplicates.join(', ').toUpperCase()}</b> in the first input field`);
  }
  if (secondDuplicates.length) {
    messageParts.push(`The error letter${secondDuplicates.length > 1 ? 's are' : ' is'} <b>${secondDuplicates.join(', ').toUpperCase()}</b> in the 2nd input field`);
  }
  duplicateLettersErrorMessage.value = messageParts.join(', and ') + '!';
};

// MAIN FUNCTION OF THIS APPLICATION USING TYPESCRIPT FUNCTIONS IN ADDITIONAL FILE wordle.ts
// USING checkForDuplicateLetters function to check for errors
const processInputWord = () => {
  // guard statement to verify a user isn't trying to exclude letters that are also included
  if (checkForDuplicateLetters(userInput.value) === true) {
    return;
  }
  invalidInput.value = false; // Remove the error message on the next submission if error is fixed!
  getOutputSuccessMessage(); // submission successful message
  // if any input is not a letter this says find exact character position matches
  if (notLetter.test(userInput.value) && !checkboxValue.value) {
    processedWords.value = lettersMatching(userInput.value, userInputExcludeLetters.value);

    // 2nd input field matches exact character position to remove the letter
    // where the user guessed the letter was in the word but it was not there  
    if (notLetter.test(userInputInWordSomewhere.value) && userInputInWordSomewhere.value.length > 0) {
      const filteredWords = processedWords.value;
      processedWords.value = lettersMatching(userInputInWordSomewhere.value, userInputExcludeLetters.value, filteredWords, true);
      //filteredWords = processedWords.value;
      let cleanedStr = userInputInWordSomewhere.value.replace(/[^a-zA-Z]/g, "").toLowerCase();

      // removes all words that do not have a misguessed letter in the word!
      let secondFilter = processedWords.value.filter(word =>
        [...cleanedStr].every(letter => word.includes(letter))
      );
      processedWords.value = secondFilter;

      return;
    }
    // if it is only letters we just search words that MUST include these letters as a secondary search
    if (userInputInWordSomewhere.value.length > 0) {
      const filteredWords = processedWords.value;
      processedWords.value = processWords(userInputInWordSomewhere.value, userInputExcludeLetters.value, filteredWords);
      return;
    }
    return;
  } else { // this else is for when the user is not trying to find exact character position matches
    processedWords.value = processWords(userInput.value, userInputExcludeLetters.value);
    const filteredWords = processedWords.value;
    if (notLetter.test(userInputInWordSomewhere.value)) {
      processedWords.value = lettersMatching(userInputInWordSomewhere.value, userInputExcludeLetters.value, filteredWords, true);
    } else {
      processedWords.value = processWords(userInputInWordSomewhere.value, userInputExcludeLetters.value, filteredWords);
    }
  }
};
</script>


<template>
  <main style="min-height: 800px; background: rgb(255, 255, 255);">
    <div class="container">
      <div class="grid-box">
        <section class="text-section">
          <div calss="center-element">
            <h1>Search assistant to help find words for the Wordle game</h1>
          </div>
          <h2>What game is this designed to help solve? &rarr;
            <a style="color: #42b983;" class="text-links"
              href="https://www.nytimes.com/games/wordle/index.html">Wordle</a>
          </h2>

          <p>
            This is a simple search tool to help find words for the Wordle game. The game is a word puzzle where you
            have
            to guess the word.
            <br>This is a Nuxt.js copy of my code from <a class="text-links"
              href="https://conradswebsite.com/projects/search-assistant-to-help-find-words-for-the-wordle-game">conradswebsite.com</a><br>
            The <a class="text-links" href="https://github.com/conrizzo/search-wordle-words/tree/main/pages">GitHub</a>
            code is here!
          </p>

          <button class="instruction-button" type="button"
            @click="isRotated = !isRotated, viewInstructions = !viewInstructions"
            style="text-align: left; display: flex; align-items: center;">
            <svg style="width: 32px; height: 32px; transition: transform 0.5s;" viewBox="0 0 32 32" fill="none"
              stroke="#42b983" stroke-width="2" :class="{ 'rotate': isRotated }">
              <polyline points="8 12 16 20 24 12" />
            </svg>

            <span v-if="!viewInstructions" class="description-span">
              View Instructions</span>
            <span v-else class="description-span">
              Close Instructions</span>

          </button>


          <transition name="slide">
            <div v-if="viewInstructions" class="instructions">

              <h3 style="padding-top: 1rem;">1st Input</h3>
              <p>
                Typing letters in the <b>Include</b> field will
                search for those letters in any position in a word.
              </p>
              <p>
                In order to search for words with a specific letter in a specific position add anything
                other than a
                letter between the letters.
              </p>
              <p>
                <b>For example:</b> Type "<b>_R_A_</b>" to search for words with the 2nd letter
                <b>R</b> and 4th letter <b>A</b>.
                Typing "<b>##EAM</b>" will search for words with the 3rd letter "<b>E</b>", 4th letter "<b>A</b>",
                and 5th letter
                "<b>M</b>".
              </p>

              <h3>2nd Input</h3>
              <p>
                The "<b>Letter in word somewhere</b>" field is a search modification.
                It can be used by itself to exclude specific letters at specific positions, or as a
                secondary filter done after the <b>Include</b> field is evaluated. Typing "<b>AAAA#</b>" here will
                exclude
                all words with <b>A</b> in the 1st-4th positions.
                If exact positions
                are used in the <b>Include</b> field with "B#R##" and
                <b>Letter in word somewhere</b> has the letter <b>C</b> entered, then the result will only be words
                with first letter B, third letter R, and the letter C somewhere in the word.
              </p>
              <p>
                As a reminder, if any character other than a letter is used in the "<b>Letter in word somewhere</b>"
                field
                then
                all words with letters in these specific positions within the 5 letter word will be removed!
                Doing this tells the search we are saying each of the letters are in the word, but were incorrect
                guesses
                at these specific
                positions, so remove the words with letters at these positions!
              </p>

              <h3>3rd Input</h3>
              <p>
                The "<b>Exclude</b>" field will exclude any words with the letters entered.
                For example, if input in this field is "AB"
                then all words with the letter "A" or "B" will be excluded. This says these letters are
                not in the word at all.
              </p>
            </div>
          </transition>
        </section>

        <form @submit.prevent="processInputWord()">
          <span style="color:#fff;">{{ outputSuccessMessage }}</span>
          <div class="submission-area">

            <label for="userInput1" class="include-label-text">Include ({{ inputLength }}):</label>
            <input id="userInput1" class="input-field-style" style="background-color: rgb(217, 255, 220);"
              placeholder="Include letters" type="text" v-model="userInput" maxlength="5" />

            <label for="userInput2" class="include-label-text">Letter in word somewhere (Optional) ({{ secondInputLength
              }}):</label>
            <input id="userInput2" class="input-field-style" style="background-color: rgb(255, 255, 172);"
              placeholder="Letter in word somewhere" type="text" v-model="userInputInWordSomewhere" maxlength="5" />

            <label for="userInput3" class="include-label-text">Exclude (Optional):</label>
            <input id="userInput3" class="input-field-exclude-letters-style"
              style="background-color: rgb(245, 245, 245);" placeholder="Exclude letters" type="text"
              v-model="userInputExcludeLetters" maxlength="18" />
            <!--
        <label>
          <span class="character-indice-font">Match Indices</span>
          <input class="check-box" type="checkbox" name="myCheckbox" v-model="checkboxValue"
            @change="handleCheckboxChange">
        </label>
        -->
          </div>
          <div class="center-element">
            <button type="submit" class="clean-button upper-spacing">
              Submit
            </button>
          </div>
          <dialog class="close" :open="invalidInput">
            <button type="button" autofocus @click="invalidInput = false;" class="close-button"></button>
            <div style="text-align: left;">
              <span class="description-span" style="color: #ff5959;">Error</span>
            </div>
            <p>You are trying to include and exclude the same data!<br>
              <span v-html="duplicateLettersErrorMessage"></span>
            </p>
          </dialog>
        </form>

      </div>
    </div>

    <!--  <img :src="(getImageSrc() as string)"> for testing -->
    <div class="center-element">
      <span style="color: rgb(18,18,18);">Words Allowed:<b>{{ processedWords.length }}</b></span>
    </div>
    <div class="letter-output-grid-box">

      <div class="center-element" v-for="(word, index) in processedWords" :key="index">
        {{ word.toUpperCase() }}
      </div>

    </div>

    <div class="center-element" style="padding-left: 1.5rem;">
      <p>
        Note: This is not an official Wordle site and all game references are property of the respective copyright
        owners.
        This is just a project to search strings with a dataset of words using JavaScript/TypeScript.
      </p>
    </div>
  </main>
</template>


<style scoped>
h3 {
  text-align: center;
}

.center-element {
  justify-content: center;
  display: flex;
}

.rotate {
  transition: transform 0.25s;
  transform: rotate(180deg) !important;
  transform-origin: center;
}

.slide-enter-active,
.slide-leave-active {
  transition: all 0.5s;
  max-height: 700px;
}

.slide-enter,
.slide-leave-to {
  max-height: 0;
  overflow: hidden;
}

.instruction-button {
  background: none;
  cursor: pointer;
  padding-right: 0.5rem;
  margin: 0;
  display: flex;
  align-items: center;

  border: none;
  border-top: 1px solid #42b983;
  border-bottom: 1px solid #42b983;
}

.instruction-button:hover {
  background-color: rgb(238, 238, 238);
}

.instruction-button:active,
.instruction-button:focus {
  outline: none;
  -webkit-tap-highlight-color: transparent;
}

.instructions {
  margin-top: 0rem;
}

.instructions p:last-child {
  padding-bottom: 0rem;
}


h1 {
  color: #42b983;
  text-transform: capitalize;
  font-size: 3.5rem;
  padding: 0rem 0rem 3rem 0rem;
  font-family: 'Signika', sans-serif;
  font-weight: 300;
  line-height: 1;
  text-align: left;
}

h2 {
  color: #42b983;
  text-transform: capitalize;
  text-align: left;
  font-size: 2.1rem;
  line-height: 1.2;
  margin-bottom: 1rem;
}

p {
  padding-bottom: 1rem;
  text-align: left;
  max-width: 100rem;
}

.description-span {
  color: #42b983;
  text-transform: capitalize;
  font-size: 1.5rem;
}


.text-section {
  margin: 0 auto;
  max-width: 38rem;
  background: rgb(255, 255, 255);
  color: rgba(18, 18, 18);
}

.grid-box {
  display: grid;
  grid-template-columns: 1fr;
  grid-gap: 1rem;
  align-items: flex-start;
  flex-direction: column;
  background: rgb(255, 255, 255);
  padding: 1.5rem 1.5rem 1.5rem 1.5rem;
  margin: 0 auto;
  max-width: 60rem;
  border-radius: 1rem;
}

.letter-output-grid-box {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  row-gap: 1rem;
  align-items: flex-end;
  background: rgb(255, 255, 255);
  color: rgb(18, 18, 18);
  padding: 1rem .25rem 1rem .25rem;
  max-width: 30rem;
  margin: 0 auto;
  border-radius: 0.5rem;
  min-height: 100px;
}

select:focus,
button:focus {
  outline: none;
}

/* make custom outline  https://stackoverflow.com/questions/16156594/how-to-change-border-color-of-textarea-on-focus */
input:focus {
  outline: none !important;
  border: 1px solid #545454;
}

.input-field-style {
  padding: 5px 1rem;
  margin: 8px 0;
  margin-left: 0.1rem;
  border-radius: 0.5em;
  border: 1px solid #ccc;
  box-sizing: border-box;
  margin-right: .5rem;
  font-size: 1.5rem;
  width: 100%;
  letter-spacing: .1rem;
  text-transform: uppercase;
}

.input-field-style::placeholder,
.input-field-exclude-letters-style::placeholder {
  text-transform: none;
  letter-spacing: 0;
}

.input-field-exclude-letters-style {
  padding: 5px 1rem;
  margin: 8px 0;
  margin-left: 0.1rem;
  border-radius: 0.5em;
  border: 1px solid #ccc;
  box-sizing: border-box;
  margin-right: .5rem;
  font-size: 1.5rem;
  width: 100%;
  /*letter-spacing: .1rem;*/
  text-transform: uppercase;
}

.exclude-div {
  display: flex;
  align-items: center;
  white-space: nowrap;
  padding-left: 2rem;
}

.container {
  padding: 2em;
}

.check-box {
  cursor: pointer;
  margin-left: 0.25rem;
  transform: scale(1.5);
}

input[type='checkbox'] {
  accent-color: #42b983;
}

.upper-spacing {
  margin-top: 1rem;
}

.upper-spacing:active {
  background: #42b983;
}

label {
  background: #ffffff;
  cursor: pointer;
  margin-left: 0.5rem;
  padding: .5rem;
  padding-right: .1rem;
  border-radius: .5rem;
  font-weight: bold;
  margin-bottom: -2rem;
}

.close-button {
  position: absolute;
  top: -2px;
  right: -2px;
  border-top-right-radius: .8rem;
  border-bottom-left-radius: 1rem;
  background: none;
  border: none;
  font-size: 1.5rem;
  cursor: pointer;
  z-index: 3;
  width: 40px;
  height: 40px;
  background-color: #ff0000;
}

.close-button:hover {
  background-color: #ff5959;
}

.close-button:before,
.close-button:after {
  position: absolute;
  content: " ";
  height: 28px;
  width: 3px;
  background-color: #000000;
  top: .33rem;
  left: 1.25rem;
}

.include-label-text {
  display: flex;
  align-items: flex-start;
  padding-bottom: 1.3rem;
  color: rgb(18, 18, 18);
  background-color: rgb(255, 255, 255);
  cursor: auto;
}

.close-button:before {
  transform: rotate(45deg);
}

.close-button:after {
  transform: rotate(-45deg);
}

dialog {
  position: relative;
  margin: 0 auto;
  border-radius: 1rem;
  padding: 1rem;
  z-index: 2;
  border: 3px solid #ff5959;
  margin-top: 1rem;
  max-width: 39.75rem;
}

.submission-area {
  border: 3px solid #42b983;
  background-color: rgb(255, 255, 255);
  max-width: 39.5rem;
  margin: 0 auto;
  border-radius: .75rem;
  padding: 0rem 1rem .5rem 1rem;
}

.character-indice-font {
  font-size: 1rem;
  padding-right: 0.2rem;
}

@media screen and (max-width: 70rem) {
  .container {
    padding: .5em;
    background: #ffffff;
  }

  .character-indice-font {
    font-size: 0.9rem;
  }


  .grid-box {
    grid-template-columns: 1fr;
    padding-left: 1rem;
    padding-right: 1rem;
  }

  h1 {
    font-size: 2.5rem;
    padding-bottom: 5rem;
  }

  h2 {
    font-size: 1.5rem;
  }
}
</style>