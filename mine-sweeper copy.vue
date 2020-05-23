<template>
  <div class='controls'>
    <div>
      <label>
        <div>Difficulty:</div>
        <input v-model='difficulty' type='range' min="0" max="4" step="1">
      </label>
      <div>{{ difficulty }}</div>
    </div>
    <div>
      <label>
        <div>Columns:</div>
        <input v-model='cols' type='range' min="2" max="50" step="1">
      </label>
      <div>{{ cols }}</div>
    </div>
    <div>
      <label>
        <div>Rows:</div>
        <input v-model='rows' type='range' min="2" max="50" step="1">
      </label>
      <div>{{ rows }}</div>
    </div>
  </div>
  <div class='grid' ref='gridRef'>
    <button 
      v-for='cell in cells' 
      :key='cell' 
      :class="['cell', cell.opened && cell.mine && '_mine', cell.opened && '_opened', cell.flaged && '_flaged']"
      :style='getNumberColor(cell.adjacent)'
      @mouseup='onMouseUp($event, cell)'
      @mousedown='onMouseDown($event)'
      @contextmenu.prevent
    >
    {{ cell.opened && !cell.mine && cell.adjacent ? cell.adjacent : '' }}
    </button>
  </div>
  <div v-if='cells.filter(c => !c.mine).every(c => c.opened)' class='win' @click='regenerateBoard'>
    <div>WIN!</div>
  </div>
</template>

<script lang="ts">
import { reactive, watch, onMounted, ref, Ref, computed } from 'vue';

interface Cell {
  index: number
  opened: boolean
  mine: boolean
  adjacent: number
  flaged: boolean
}

export default {
  name: 'MineSweeper',
  setup() {
    const gridRef = ref(null)
    const difficulty = ref(3)
    const difficulties = [0.95, 0.90, 0.85, 0.80, 0.75]
    const realDifficulty = computed(() => difficulties[difficulty.value])
    const cols = ref(30)
    const rows = ref(16)
    const pressedMouseButtons = ref(new Set())

    function regenerateBoard() {
      cells.splice(0, cells.length)
      cells.push(...generateCells(cols.value, rows.value, realDifficulty))
    }

    function onDimensionsChange() {
      setGrid()
      regenerateBoard()
    }

    function regenerateBoardOnChange(r: Ref<number>) {
      return (v) => {
        if (!Number.isInteger(v)) {
          r.value = +v
        } else {
          onDimensionsChange()
        }
      }
    }

    watch(cols, regenerateBoardOnChange(cols))
    watch(rows, regenerateBoardOnChange(rows))
    watch(difficulty, regenerateBoardOnChange(difficulty))

    let cells: Array<Cell> = reactive(generateCells(cols.value, rows.value, realDifficulty))

    function onMouseDown(e: MouseEvent) {
      pressedMouseButtons.value.add(e.which)
    }

    function reveal(cell: Cell) {
      cell.opened = true

      if (cell.flaged) cell.flaged = false

      if (!cell.adjacent && !cell.mine) {
        getNeighbourIndexies(cell.index, cols.value, rows.value)
          .forEach(i => {
            const cell = cells[i]
            if (!cell.opened) reveal(cell)
          })
      }
    }

    function onMouseUp(e: MouseEvent, cell: Cell) {
      setTimeout(() => {
        pressedMouseButtons.value.delete(e.which)
      })

      if (cell.opened 
      && ((pressedMouseButtons.value.has(1) && e.which === 3)
      || (pressedMouseButtons.value.has(3) && e.which === 1))) {
        const neighbours = getNeighbourIndexies(cell.index, cols.value, rows.value)
        const allMinesMarked = neighbours
          .reduce((acc, i) => {
            if (!acc) return false
            return (!cells[i].mine && !cells[i].flaged) || (cells[i].mine && cells[i].flaged)
          }, true)
        if (allMinesMarked) {
          neighbours.forEach(i => {
            if(!cells[i].mine) {
              reveal(cells[i])
            }
          })
        }
        return
      }

      if (e.which === 3 && !cell.opened && !pressedMouseButtons.value.has(1)) {
        cell.flaged = !cell.flaged
        return
      }

      const mines = cells
        .filter(c => c.mine)

      if (mines.every(c => c.opened)) {
        regenerateBoard()
        return
      }

      if (!cell.opened && !cell.flaged) {
        if (cell.mine) {
          mines
            .forEach(c => {
              cells[c.index].opened = true
            })
          return
        }

        reveal(cell)
      }
    }

    function onCellRightClick(cell) {
      if (!cell.opened) {
        cell.flaged = !cell.flaged
      }
    }

    function setGrid() {
      gridRef.value.style.setProperty('--cols', cols.value);
      gridRef.value.style.setProperty('--rows', rows.value);
    }

    onMounted(setGrid)

    return {
      cells,
      onMouseUp,
      onMouseDown,
      onCellRightClick,
      getNumberColor,
      gridRef,
      difficulty,
      rows,
      cols,
      regenerateBoard
    }
  }
}

function getNeighbourIndexies(index: number, cols: number, rows: number) {
  const arr = []
  const remainder = index % cols
  arr.push(index - cols)
  arr.push(index + cols)

  if (remainder - 1 >= 0) {
    const left = index - 1
    ;[left - cols, left, left + cols].forEach(i => {
      arr.push(i)
    })
  }

  if (remainder + 1 !== cols) {
    const right = index + 1
    ;[right - cols, right, right + cols].forEach(i => {
      arr.push(i)
    })
  }

  const cellsAmount = rows * cols
  return arr.filter(i => i >= 0 && i < cellsAmount)
}

function generateCells(cols: number, rows: number, difficulty: Ref<number>) {
  return new Array(cols * rows)
    .fill(0)
    .map((_, index) => ({
      index,
      opened: false,
      mine: Math.random() > difficulty.value,
      adjacent: 0,
      flaged: false
    }))
    .map((cell, index, array) => {
      cell.adjacent = getNeighbourIndexies(index, cols, rows)
        .reduce((amount, index) => {
          if(array[index].mine) {
            amount += 1
          }

          return amount
        }, 0)

      return cell
    })
}

function getNumberColor(n) {
  return `color: ${[
    'blue',
    'green',
    'red',
    'darkblue',
    'darkred',
    'lightseagreen',
    'black',
    'grey'
  ][n - 1]};`
}
</script>

<style>
  .grid {
    --cell-size: min(calc(100vw / var(--cols)), calc((100vh - 30px) / var(--rows)));
    display: grid;
    grid-template-columns: repeat(var(--cols), var(--cell-size));
    grid-template-rows: repeat(var(--rows), var(--cell-size));
    border-left: 1px solid grey;
    border-top: 1px solid grey;
  }

  .cell {
    border-right: 1px solid grey;
    border-bottom: 1px solid grey;
    background-color: white;
    font-size: calc(var(--cell-size) * 0.8);
    line-height: calc(var(--cell-size) * 0.8);
    text-align: center;
    font-weight: bold;
  }

  .cell._opened {
    background-color: lightgrey;
  }

  .cell._mine {
    background-color: black;
  }

  .cell._flaged {
    background-color: pink;
  }

  .controls {
    display: flex;
    flex-wrap: wrap;
  }

  .controls > div {
    display: flex;
    align-items: center;
    margin-right: 20px;
  }

  .controls > div > label {
    margin-right: 10px;
    display: flex;
    align-items: center;
  }

  .win {
    position: absolute;
    width: 100vw;
    height: 100vh;
    left: 0;
    top: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: white;
    color: red;
    font-weight: 700;
    font-size: 50vmin;
  }
</style>
