<script setup lang="ts">
import { reactive } from 'vue'
import { Button } from '@/components/ui/button'
import Dict from '@/dict'

interface Letter {
  value: string,
  guessed: boolean,
}

const StateEnum = {
  Ongoing: 'ongoing',
  Won: 'won',
  Lost: 'lost',
} as const
type State = (typeof StateEnum)[keyof typeof StateEnum]

interface Hangman {
  word: string,
  dashes: string,
  alphabet: Letter[],

  guess: number,
  image: string,
  state: State,
}

const hangman = reactive<Hangman>({
  word: '',
  dashes: '',
  alphabet: [],

  guess: -1,
  image: '',
  state: StateEnum.Ongoing,
})

function setImage(s: string) {
  hangman.image = new URL(`./assets/${s}.png`, import.meta.url).href
}

function setup() {
  hangman.word = Dict.randomWord()
  hangman.dashes = hangman.word.replace(/[A-Z]/g, '_')
  hangman.alphabet = []
  const alphabet = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'.split('')
  alphabet.forEach(letter => {
    hangman.alphabet.push({
      value: letter,
      guessed: false,
    })
  })

  hangman.guess = 0
  setImage(`${hangman.guess}`)
  hangman.state = StateEnum.Ongoing
}

function guess(letter: Letter) {
  if (hangman.state != StateEnum.Ongoing)
    return

  letter.guessed = true
  const found = hangman.word.includes(letter.value)
  if (!found) {
    hangman.guess = hangman.guess + 1
    setImage(`${hangman.guess}`)
    if (hangman.guess === 9) {
      hangman.state = StateEnum.Lost
      hangman.dashes = hangman.word
    }

    return
  }

  const dashes = hangman.dashes.split('')
  for (let i = 0; i < hangman.word.length; i++) {
    if (hangman.word[i] == letter.value) {
      dashes[i] = letter.value
    }
  }

  hangman.dashes = dashes.join('')
  if (!hangman.dashes.includes('_')) {
    hangman.state = StateEnum.Won
    setImage('win')
  }
}

setup()
console.log(hangman.word)
</script>

<template>
  <p class="mt-8 mb-16 text-4xl font-mono font-bold tracking-[.33em]">
    {{ hangman.dashes }}
  </p>
  <div class="h-80 flex justify-center">
    <img :src="hangman.image" alt="hangman" class="object-contain" />
  </div>

  <div class="h-8 my-16">
    <Button v-if="hangman.state != StateEnum.Ongoing" @click="setup()">
      Reset
    </Button>
    <Button v-else variant="secondary">
      Reset
    </Button>
  </div>

  <div class="grid grid-cols-9 md:grid-cols-13 gap-1">
    <Button v-for="letter in hangman.alphabet" @click="!letter.guessed && guess(letter)"
      :variant="!letter.guessed ? 'default' : 'secondary'" class="h-10 md:h-15 text-xl font-bold">
      {{ letter.value }}
    </Button>
  </div>
</template>

<style scoped></style>
