<script setup lang="ts">
import { ref, onMounted, computed } from 'vue'
import type { MaybeElement } from '@vueuse/core'
import { useAnimate } from '@vueuse/core'
// import { stringify } from '@vueuse/docs-utils'
import ConfettiExplosion from "vue-confetti-explosion";
import { reactive, shallowRef } from 'vue'

const el = shallowRef<MaybeElement>()


const {
  reverse,
  startTime,
  currentTime,
  playbackRate,
  playState,
  replaceState,
  pending,
} = useAnimate(
  el,
  [
    { clipPath: 'circle(20% at 0% 30%)' },
    { clipPath: 'circle(20% at 50% 80%)' },
    { clipPath: 'circle(20% at 100% 30%)' },
  ],
  {
    duration: 3000,
    iterations: 2,
    direction: 'alternate',
    easing: 'cubic-bezier(0.46, 0.03, 0.52, 0.96)',
  },
)

const text = JSON.stringify(reactive({
  startTime,
  currentTime,
  playbackRate,
  playState,
  replaceState,
  pending,
}))


const board = ref(Array(9).fill(null))
const winningCombination = ref<number[]>([])
const player = ref(true)
const winner = ref('')
const hasWinner = ref(false)
const handleClick = (index: number) => {
  if(!board.value[index]) {
    if(player.value)
      board.value[index] = '🍆'
    else 
      board.value[index] = '💦'

    
    if(checkWin()) {
      hasWinner.value = true
      winner.value = player.value? "X" : "O"
    }
    console.log(checkDraw())
    if(checkDraw()) {
      winningCombination.value = Array.from({ length: 9 }, (_, i) => i)
      setTimeout(() => clearWinningCells(), 500)
    }
    player.value = !player.value
  } else {
    console.log("select another box")
  }
}

const clearWinningCells = () => {
  winningCombination.value.forEach((index, i) => {
    setTimeout(() => {
      const cell = document.querySelectorAll('.grid-cell')[index]
      if (cell) {
        cell.classList.add('explode')
      }

      setTimeout(() => {
        board.value[index] = null
        if (i === winningCombination.value.length - 1) {
          winningCombination.value = []
        }
      }, 300)
    }, i * 300)
  })
}

const checkWin = () => {
  const winningCombinations = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [2, 4, 6],
    [0, 4, 8],
  ]

  for (let combination of winningCombinations) {
    if (
      board.value[combination[0]] &&
      board.value[combination[0]] === board.value[combination[1]] &&
      board.value[combination[1]] === board.value[combination[2]]
    ) {
      winningCombination.value = combination
      setTimeout(() => clearWinningCells(), 500)
      return true
    }
  }
  return false
}

const checkDraw = () => {
  return board.value.every((cell) => cell!== null);
}

const resetBoard = () => {
  board.value = Array(9).fill(null)
  winner.value = ''
  player.value = true
  hasWinner.value = false
  winningCombination.value = []
}

onMounted( () => {
  reverse()
})
</script>

<template>
  <div class="min-h-screen flex flex-col items-center justify-center bg-slate-200">
    <h1 ref="el" class="text-6xl font-bold text-center pb-1.5">Tik Tik Toe</h1>
    <ConfettiExplosion v-if="hasWinner" :particleCount="300" :force="0.5" :colors="['var(--yellow)', 'var(--red)', '#000']" />
    <p v-if="hasWinner" class="text-2xl font-bold text-center pb-1.5">Player {{ winner }} wins</p>
    <div class="grid grid-cols-3 gap-4 mb-4">
      <div 
        v-for="(cell, index) in board" 
        :key="index" 
        :class="[
          'grid-cell w-24 h-24 border border-gray-400 p-2 flex items-center justify-center',
          { 'bg-blue-500': winningCombination.includes(index) }
        ]"
      >
        <button 
          @click="handleClick(index)" 
          class="w-24 h-24 text-3xl"
        >
          {{cell}}
        </button>
      </div>
    </div>
    <button @click="resetBoard" class="w-24 h-12 text-xl font-bold text-center mt-4">Reset</button>
    <p class="text-sm text-gray-600">@krobuncal</p>
  </div>
</template>

<style>
/* Explosion animation */
@keyframes explode {
  0% {
    transform: scale(1);
    opacity: 1;
  }
  50% {
    transform: scale(1.5);
    opacity: 0.7;
  }
  100% {
    transform: scale(0);
    opacity: 0;
  }
}

/* Add explosion effect */
.explode {
  animation: explode 0.3s ease-out;
  animation-fill-mode: forwards;
}

/* Highlight winning cells */
.bg-blue-500 {
  background-color: #4299e1; /* Tailwind's blue-500 */
}
</style>