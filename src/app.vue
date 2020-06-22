<template>
  <nav>
    <button @click="current = Games.Minesweeper">Minesweeper</button>
    <button @click="current = Games.TicTacToe">Tic-Tac-Toe</button>
  </nav>
  <Minesweeper v-if="current === Games.Minesweeper"></Minesweeper>
  <TicTacToe v-else-if="current === Games.TicTacToe"></TicTacToe>
</template>

<script lang="ts">
import Minesweeper from './components/minesweeper.vue'
import TicTacToe from './components/tic-tac-toe.vue'
import { ref } from 'vue'

enum Games {
  Minesweeper = 'Minesweeper',
  TicTacToe = 'TicTacToe'
}

export default {
  name: 'App',
  components: {
    Minesweeper,
    TicTacToe
  },
  setup() {
    const current = ref(Games.TicTacToe)

    return {
      current,
      Games
    }
  }
}
</script>
