<script setup lang="ts">
import { computed, reactive, ref, watch } from 'vue'
import GameLetter from './components/GameLetter.vue'

const abodes: Record<string, string> = { M: 'Mansion', A: 'Apartment', S: 'Shack', H: 'House' }
const questions = [
  'Who will you marry?',
  'What will be your salary?',
  'How many kids?',
  'How long will you live?'
]
const elementCount = questions.length * 4 + 4
const questionRanges = [3, 7, 11, 15, 19]
let results: string[] = []
let gameTimer: number
let chosenNumber: number = 0
let discardIndex = 0
let gameStarted = false
let chosenElements: number[] = []
let selectedIndex = ref(-1)

const options = reactive([
  [
    { discarded: false, value: 'M' },
    { discarded: false, value: 'A' },
    { discarded: false, value: 'S' },
    { discarded: false, value: 'H' }
  ], //mash
  [
    { discarded: false, value: 'Riana' },
    { discarded: false, value: 'Alice Cooper' },
    { discarded: false, value: 'Bob Fossil' },
    { discarded: false, value: 'Brian Ferry' }
  ], //q1
  [
    { discarded: false, value: '1000' },
    { discarded: false, value: '1m' },
    { discarded: false, value: '1p' },
    { discarded: false, value: '20000' }
  ], //q2
  [
    { discarded: false, value: '1' },
    { discarded: false, value: '2' },
    { discarded: false, value: '3' },
    { discarded: false, value: '500' }
  ], //q3
  [
    { discarded: false, value: '70' },
    { discarded: false, value: '85' },
    { discarded: false, value: '100' },
    { discarded: false, value: '5000' }
  ] //q4
])

const getNextSelectedIndex = (currentIndex: number) => {
  while (chosenElements.sort((a, b) => a - b).includes(currentIndex)) {
    currentIndex += 1
    if (currentIndex >= elementCount) {
      currentIndex = 0
    }
  }
  return currentIndex
}

const getDiscardedOptionCount = (optionSection: { discarded: boolean }[]) =>
  optionSection.filter((option) => option.discarded).length

const startGame = (event: Event) => {
  event.preventDefault()
  chosenNumber = Math.floor(Math.random() * (7 - 3 + 1)) + 3
  gameStarted = true
  selectedIndex.value = -1

  gameTimer = setInterval(() => {
    selectedIndex.value += 1
    if (selectedIndex.value >= elementCount) {
      selectedIndex.value = 0
    }
    selectedIndex.value = getNextSelectedIndex(selectedIndex.value)

    const optionIndex = questionRanges.findIndex((range) => selectedIndex.value <= range)
    const optionSection = options[optionIndex]
    const answerIndex = selectedIndex.value % 4

    if (!optionSection[answerIndex].discarded) {
      discardIndex += 1
    }

    if (discardIndex === chosenNumber) {
      chosenElements.push(selectedIndex.value)
      discardIndex = 0
      optionSection[answerIndex].discarded = true
    }

    // calculate discard count, if its 3, add the remaining non-discarded element to chosenElements
    const discardedCount = getDiscardedOptionCount(optionSection)
    if (discardedCount === 3) {
      const remainingElementIndex = optionSection.findIndex((option) => !option.discarded)
      chosenElements.push(selectedIndex.value + (remainingElementIndex - answerIndex))
    }
  }, 450)
}

// compute the value of every option, if all but one are discarded, the game is over
const gameComplete = computed(() =>
  options.every((option) => option.filter((answer) => answer.discarded).length === 3)
)

watch(gameComplete, (gameIsComplete) => {
  if (gameIsComplete) {
    clearInterval(gameTimer)
    selectedIndex.value = -1
    results = options.map((option) => option.find((answer) => !answer.discarded)?.value ?? '')
  }
})

const reset = () => {
  clearInterval(gameTimer)
  selectedIndex.value = -1
  gameStarted = false
  chosenNumber = 0
  discardIndex = 0
  results = []
  chosenElements = []
  options.forEach((option) => {
    option.forEach((answer) => {
      answer.discarded = false
    })
  })
}
</script>

<template>
  <header>
    <h1>
      <GameLetter
        v-for="(letter, index) in options[0]"
        :key="index"
        :gameIndex="index"
        :selectedIndex
        :discarded="letter.discarded"
        >{{ letter.value }}</GameLetter
      >
    </h1>
    <p class="number" v-if="chosenNumber > 0">You chose a {{ chosenNumber }}</p>
  </header>

  <main>
    <form @submit="startGame">
      <div class="questions">
        <div v-for="(question, questionIndex) in questions" :key="question">
          <h2>{{ question }}</h2>
          <ul>
            <li
              v-for="({ discarded }, answerIndex) in options[questionIndex + 1]"
              :key="answerIndex"
            >
              <input
                type="text"
                :disabled="gameStarted"
                required
                v-model="options[questionIndex + 1][answerIndex].value"
                :class="{
                  active: selectedIndex === questionIndex * 4 + 4 + answerIndex,
                  discarded
                }"
              />
            </li>
          </ul>
        </div>
      </div>
      <div class="controls">
        <button type="submit" :disabled="gameStarted">Play</button>
        <button type="button" class="reset" @click="reset">Reset</button>
      </div>
    </form>
    <Transition name="slide-fade">
      <div class="results" v-if="gameComplete">
        <h2>Your fortune</h2>
        <p>You will live in {{ results[0] === 'A' ? 'an' : 'a' }} {{ abodes[results[0]] }}</p>
        <ul>
          <li v-for="(question, index) in questions" :key="question">
            <h3>{{ question }}</h3>
            <p>{{ results[index + 1] }}</p>
          </li>
        </ul>
        <button type="button" @click="reset">Play again</button>
      </div>
    </Transition>
  </main>
</template>

<style scoped>
header {
  text-align: center;
  h1 span {
    font-size: 3rem;
    font-weight: bold;
  }

  .number {
    color: white;
    margin-bottom: 1rem;
  }
}

main {
  position: relative;
  background: grey;
  padding: 2rem;
  border-radius: 2rem;
}
h2 {
  color: var(--secondary-color);
  font-weight: bold;
}
.questions {
  display: block;

  @media screen and (min-width: 768px) {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    grid-template-rows: auto auto;
    gap: 1rem;
  }
  gap: 1rem;
  width: 100%;
}
.controls {
  display: flex;
  align-items: center;
  gap: 1rem;
  justify-content: center;
  margin-top: 1rem;
  width: 100%;
}
button {
  width: 25rem;
  padding: 1rem;
  font-size: 1.3rem;
  font-weight: bold;
  background-color: var(--primary-color);
  color: black;
  border: none;
  border-radius: 1rem;
  cursor: pointer;
  box-shadow: 0 0 1rem 0 rgba(0, 0, 0, 0.2);

  &:hover {
    background-color: var(--primary-color-dark);
  }

  &.reset {
    background-color: var(--alert-color);
    &:hover {
      background-color: var(--alert-color-dark);
    }
  }

  &[disabled] {
    background-color: darkgrey;
    cursor: auto;
  }
}

ul {
  list-style-type: none;
  padding: 0;
}
li {
  margin-bottom: 1rem;
}
input {
  width: 100%;
  padding: 0.5rem;
  border-radius: 5px;
  border: 2px solid white;
  transition: all 0.3s ease;

  &[disabled] {
    background-color: white;
  }

  &.active {
    border-color: var(--secondary-color);
    box-shadow: 0 0 1rem 0.5rem var(--secondary-color);
  }

  &.discarded {
    opacity: 0.2;
  }
}

.results {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 75%;
  transform: translate(-50%, -50%);
  background: white;
  padding: 2rem;
  border-radius: 2rem;
  color: black;
  border: 1px solid grey;
  box-shadow: 0 0 1rem 0 rgba(0, 0, 0, 0.5);

  h2 {
    font-size: 2.5rem;
    color: darkgreen;
  }

  p {
    font-size: 1.5rem;
  }
  button {
    width: 100%;
  }
}

.slide-fade-enter-active,
.slide-fade-leave-active {
  transition: all 0.5s ease-out;
}

.slide-fade-enter-from,
.slide-fade-leave-to {
  opacity: 0;
}
</style>
