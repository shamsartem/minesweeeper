<template>
  <div :class="[c.grid, gameState === GameStates.PlayerXWon && c._playerXWon, gameState === GameStates.PlayerOWon && c._playerOWon]">
    <div 
      v-for="(cell, i) in cells" 
      :key="cell"
      :class="[c.cell, cell === CellStates.PlayerX && c._playerX, cell === CellStates.PlayerO && c._playerO]"
      @click="handleCellClick(i)"
    >
    </div>
  </div>
</template>

<script lang="ts">
import { reactive, ref } from 'vue'

enum Players {
  PlayerX = 'PlayerX',
  PlayerO = 'PlayerO'
}

enum CellStates {
  Empty = 'Empty',
  PlayerX = 'PlayerX',
  PlayerO = 'PlayerO'
}

enum GameStates {
  InProgress = 'InProgress',
  PlayerXWon = 'PlayerXWon',
  PlayerOWon = 'PlayerOWon',
  Draw = 'Draw'
}

export default {
  name: 'TicTacToe',
  setup() {
    const cells = reactive(generateNewCells())
    const currentPlayer = ref(Players.PlayerX)
    const gameState = ref(GameStates.InProgress)

    function handleCellClick(i) {
      if (gameState.value !== GameStates.InProgress) {
        cells.splice(0, cells.length)
        cells.push(...generateNewCells())
        gameState.value = GameStates.InProgress
        console.log(i)
        return
      }

      console.log(i, cells)

      if (cells[i] === CellStates.Empty) {
        cells[i] = currentPlayer.value
        const rows = [cells.slice(0,3), cells.slice(3,6), cells.slice(6)]
        const cols = [rows.map(r => r[0]), rows.map(r => r[1]), rows.map(r => r[2])]
        const diagonals = [[cells[0], cells[4], cells[8]], [cells[2], cells[4], cells[6]]]

        function playerWon(player) {
          function every(player) {
            return array => array.every(v => v === player)
          }

          return rows.some(every(player)) 
          || cols.some(every(player))
          || diagonals.some(every(player))
        }
  
        if (playerWon(Players.PlayerX)) {
          gameState.value = GameStates.PlayerXWon
        } else if (playerWon(Players.PlayerO)) {
          gameState.value = GameStates.PlayerOWon
        } else if (cells.every(c => c !== CellStates.Empty)) {
          gameState.value = GameStates.Draw
        }

        currentPlayer.value = currentPlayer.value === Players.PlayerX ?
          Players.PlayerO : Players.PlayerX
      }
    }

    return {
      cells,
      CellStates,
      Players,
      currentPlayer,
      handleCellClick,
      gameState,
      GameStates,
    }
  }
}

function generateNewCells() {
  return new Array(9).fill(CellStates.Empty)
}
</script>

<style module="c">
  .grid {
    --x-size: calc((100vmin / 3) - 20px);
    --o-size: calc((100vmin / 3) - 50px);
    display: grid;
    width: 100%;
    height: calc(100vh - 18px);
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat(3, 1fr);
    gap: 2px;
    background-color: gray;
  }

  .grid._playerXWon {
    background-color: red;
  }

  .grid._playerOWon {
    background-color: blue;
  }

  .cell {
    background-color: white;
    display: flex;
    justify-content: center;
    align-items: center;
    position: relative;
  }

  .cell._playerX::before {
    content: '';
    width: var(--x-size);
    height: 20px;
    background-color: red;
    transform: rotate(-45deg);
    position: absolute;
  }

  .cell._playerX::after {
    content: '';
    width: var(--x-size);
    height: 20px;
    background-color: red;
    transform: rotate(45deg);
  }

  .cell._playerO::before {
    content: '';
    width: var(--o-size);
    height: var(--o-size);
    border-radius: 9999px;
    border: 20px solid blue;
  }
</style>
