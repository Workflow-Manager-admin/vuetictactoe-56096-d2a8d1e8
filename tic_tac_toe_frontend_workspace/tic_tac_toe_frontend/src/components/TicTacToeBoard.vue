<script setup lang="ts">
import { ref, computed } from 'vue'

const EMPTY = ''
const PLAYER_X = 'X'
const PLAYER_O = 'O'

const board = ref<Array<Array<string>>>([
  [EMPTY, EMPTY, EMPTY],
  [EMPTY, EMPTY, EMPTY],
  [EMPTY, EMPTY, EMPTY]
])
const currentPlayer = ref(PLAYER_X)
const winner = ref<string | null>(null)
const isDraw = ref(false)

function resetGame() {
  board.value = [
    [EMPTY, EMPTY, EMPTY],
    [EMPTY, EMPTY, EMPTY],
    [EMPTY, EMPTY, EMPTY]
  ]
  currentPlayer.value = PLAYER_X
  winner.value = null
  isDraw.value = false
}

// PUBLIC_INTERFACE
function handleCellClick(row: number, col: number) {
  if (board.value[row][col] !== EMPTY || winner.value) return
  board.value[row][col] = currentPlayer.value
  checkWinner()
  if (!winner.value && !hasEmptyCell()) {
    isDraw.value = true
  }
  if (!winner.value && !isDraw.value) {
    currentPlayer.value = currentPlayer.value === PLAYER_X ? PLAYER_O : PLAYER_X
  }
}

function checkWinner() {
  const lines = [
    // Rows
    [ [0,0], [0,1], [0,2] ],
    [ [1,0], [1,1], [1,2] ],
    [ [2,0], [2,1], [2,2] ],
    // Columns
    [ [0,0], [1,0], [2,0] ],
    [ [0,1], [1,1], [2,1] ],
    [ [0,2], [1,2], [2,2] ],
    // Diagonals
    [ [0,0], [1,1], [2,2] ],
    [ [0,2], [1,1], [2,0] ]
  ]
  for (const line of lines) {
    const [a, b, c] = line
    const v1 = board.value[a[0]][a[1]]
    const v2 = board.value[b[0]][b[1]]
    const v3 = board.value[c[0]][c[1]]
    if (v1 && v1 === v2 && v1 === v3) {
      winner.value = v1
      break
    }
  }
}

function hasEmptyCell() {
  return board.value.some(row => row.some(cell => cell === EMPTY))
}

const statusMessage = computed(() => {
  if (winner.value) return `Winner: ${winner.value}`
  if (isDraw.value) return "It's a draw!"
  return `Turn: ${currentPlayer.value}`
})
</script>

<template>
  <div class="ttt-outer">
    <div class="ttt-status">{{ statusMessage }}</div>
    <div class="ttt-board">
      <div v-for="(row, rowIndex) in board" :key="rowIndex" class="ttt-row">
        <button
          v-for="(cell, colIndex) in row"
          :key="colIndex"
          class="ttt-cell"
          :disabled="!!cell || Boolean(winner)"
          @click="handleCellClick(rowIndex, colIndex)"
          :aria-label="cell ? `Tile ${cell}` : `Empty`"
        >
          {{ cell }}
        </button>
      </div>
    </div>
    <button class="ttt-reset" @click="resetGame">Reset</button>
  </div>
</template>

<style scoped>
.ttt-outer {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 80vh;
}

.ttt-status {
  margin-bottom: 1.5rem;
  font-size: 1.25rem;
  font-weight: 500;
  color: #35495e;
  letter-spacing: 0.2px;
}

.ttt-board {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
  background: #fff;
  border-radius: 16px;
  box-shadow: 0 2px 12px 0 rgba(53, 73, 94, 0.10);
  padding: 1rem;
}

.ttt-row {
  display: flex;
  gap: 0.25rem;
}

.ttt-cell {
  width: 68px;
  height: 68px;
  background: #f9fdfa;
  border: 2px solid #41b883;
  border-radius: 10px;
  font-size: 2.1rem;
  font-weight: 700;
  color: #35495e;
  cursor: pointer;
  outline: none;
  transition: background 0.12s, box-shadow 0.12s;
  box-shadow: 0 1px 2px rgba(53,73,94,.07);
}
.ttt-cell:hover:enabled,
.ttt-cell:focus-visible:enabled {
  background: #fffcdf;
  border-color: #ffcc00;
  box-shadow: 0 4px 18px 0 rgba(255, 204, 0, 0.17);
}
.ttt-cell:disabled {
  cursor: default;
  opacity: 0.60;
  background: #f4f4f4;
}

.ttt-reset {
  margin-top: 2rem;
  background: #ffcc00;
  color: #35495e;
  border: none;
  border-radius: 6px;
  padding: 0.8em 2.1em;
  font-size: 1.1rem;
  font-weight: 700;
  cursor: pointer;
  transition: background 0.2s, color 0.2s;
  box-shadow: 0 1px 6px rgba(53,73,94,.08);
}
.ttt-reset:hover,
.ttt-reset:focus-visible {
  background: #41b883;
  color: #fff;
}

@media (max-width: 600px) {
  .ttt-board {
    padding: 0.5rem;
  }
  .ttt-cell {
    width: 54px;
    height: 54px;
    font-size: 1.5rem;
  }
  .ttt-reset{
    width: 100%;
  }
}
</style>
