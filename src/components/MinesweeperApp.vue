<template>
  <div class="minesweeper-app">
    <div class="game-header">
      <div class="difficulty-selector">
        <v-btn-group>
          <v-btn
            v-for="level in difficultyLevels"
            :key="level.name"
            :color="currentDifficulty.name === level.name ? 'primary' : 'grey'"
            @click="setDifficulty(level)"
          >
            {{ level.name }}
          </v-btn>
        </v-btn-group>
      </div>

      <div class="game-stats">
        <div class="mine-counter">
          <v-icon color="error">mdi-bomb</v-icon>
          {{ remainingMines }}
        </div>
        <div class="timer">
          <v-icon color="primary">mdi-timer</v-icon>
          {{ formatTime(time) }}
        </div>
      </div>
    </div>

    <div class="grid-container" :style="gridStyle">
      <div
        v-for="(cell, index) in grid"
        :key="index"
        class="grid-cell"
        :class="{
          'revealed': cell.revealed,
          'mine': cell.isMine && gameOver,
          'flagged': cell.flagged
        }"
        @click="revealCell(index)"
        @contextmenu.prevent="flagCell(index)"
      >
        <template v-if="cell.revealed">
          <span v-if="cell.isMine" class="mine-icon">
            <v-icon color="error">mdi-bomb</v-icon>
          </span>
          <span v-else-if="cell.neighborMines > 0" :class="'number-' + cell.neighborMines">
            {{ cell.neighborMines }}
          </span>
        </template>
        <template v-else-if="cell.flagged">
          <v-icon color="warning">mdi-flag</v-icon>
        </template>
      </div>
    </div>

    <div class="game-controls">
      <v-btn
        color="primary"
        @click="startNewGame"
        :disabled="isPlaying"
      >
        <v-icon start>mdi-refresh</v-icon>
        Nuevo Juego
      </v-btn>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

const difficultyLevels = [
  { name: 'Principiante', rows: 9, cols: 9, mines: 10 },
  { name: 'Medio', rows: 16, cols: 16, mines: 40 },
  { name: 'Avanzado', rows: 16, cols: 30, mines: 99 }
]

const currentDifficulty = ref(difficultyLevels[0])
const grid = ref([])
const isPlaying = ref(false)
const gameOver = ref(false)
const time = ref(0)
const timer = ref(null)
const remainingMines = ref(0)

const gridStyle = computed(() => ({
  gridTemplateColumns: `repeat(${currentDifficulty.value.cols}, 1fr)`,
  gridTemplateRows: `repeat(${currentDifficulty.value.rows}, 1fr)`
}))

const initializeGrid = () => {
  const { rows, cols, mines } = currentDifficulty.value
  grid.value = []
  remainingMines.value = mines

  // Create empty grid
  for (let i = 0; i < rows * cols; i++) {
    grid.value.push({
      isMine: false,
      revealed: false,
      flagged: false,
      neighborMines: 0
    })
  }

  // Place mines randomly
  let minesPlaced = 0
  while (minesPlaced < mines) {
    const randomIndex = Math.floor(Math.random() * grid.value.length)
    if (!grid.value[randomIndex].isMine) {
      grid.value[randomIndex].isMine = true
      minesPlaced++
    }
  }

  // Calculate neighbor mines
  for (let i = 0; i < rows; i++) {
    for (let j = 0; j < cols; j++) {
      const index = i * cols + j
      if (!grid.value[index].isMine) {
        grid.value[index].neighborMines = countNeighborMines(i, j)
      }
    }
  }
}

const countNeighborMines = (row, col) => {
  const { rows, cols } = currentDifficulty.value
  let count = 0

  for (let i = -1; i <= 1; i++) {
    for (let j = -1; j <= 1; j++) {
      const newRow = row + i
      const newCol = col + j
      if (newRow >= 0 && newRow < rows && newCol >= 0 && newCol < cols) {
        const index = newRow * cols + newCol
        if (grid.value[index].isMine) count++
      }
    }
  }

  return count
}

const revealCell = (index) => {
  if (gameOver.value || grid.value[index].flagged) return

  if (!isPlaying.value) {
    startGame()
  }

  if (grid.value[index].isMine) {
    gameOver = true
    revealAllMines()
    clearInterval(timer.value)
    return
  }

  revealEmptyCells(index)
  checkWin()
}

const revealEmptyCells = (index) => {
  const { rows, cols } = currentDifficulty.value
  const row = Math.floor(index / cols)
  const col = index % cols

  if (grid.value[index].revealed || grid.value[index].flagged) return

  grid.value[index].revealed = true

  if (grid.value[index].neighborMines === 0) {
    for (let i = -1; i <= 1; i++) {
      for (let j = -1; j <= 1; j++) {
        const newRow = row + i
        const newCol = col + j
        if (newRow >= 0 && newRow < rows && newCol >= 0 && newCol < cols) {
          revealEmptyCells(newRow * cols + newCol)
        }
      }
    }
  }
}

const flagCell = (index) => {
  if (gameOver.value || grid.value[index].revealed) return

  if (!isPlaying.value) {
    startGame()
  }

  grid.value[index].flagged = !grid.value[index].flagged
  remainingMines.value += grid.value[index].flagged ? -1 : 1
  checkWin()
}

const revealAllMines = () => {
  grid.value.forEach(cell => {
    if (cell.isMine) cell.revealed = true
  })
}

const checkWin = () => {
  const win = grid.value.every(cell => 
    (cell.isMine && cell.flagged) || (!cell.isMine && cell.revealed)
  )

  if (win) {
    gameOver.value = true
    clearInterval(timer.value)
    alert('¡Felicidades! Has ganado!')
  }
}

const startGame = () => {
  isPlaying.value = true
  gameOver.value = false
  time.value = 0
  timer.value = setInterval(() => {
    time.value++
  }, 1000)
}

const startNewGame = () => {
  clearInterval(timer.value)
  isPlaying.value = false
  gameOver.value = false
  time.value = 0
  initializeGrid()
}

const setDifficulty = (level) => {
  currentDifficulty.value = level
  startNewGame()
}

const formatTime = (seconds) => {
  const minutes = Math.floor(seconds / 60)
  const remainingSeconds = seconds % 60
  return `${minutes.toString().padStart(2, '0')}:${remainingSeconds.toString().padStart(2, '0')}`
}

onMounted(() => {
  initializeGrid()
})

onUnmounted(() => {
  clearInterval(timer.value)
})
</script>

<style scoped>
.minesweeper-app {
  padding: 20px;
  max-width: 1200px;
  margin: 0 auto;
}

.game-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.game-stats {
  display: flex;
  gap: 20px;
  font-size: 1.2em;
}

.grid-container {
  display: grid;
  gap: 2px;
  background-color: #f0f0f0;
  padding: 10px;
  border-radius: 8px;
  margin-bottom: 20px;
}

.grid-cell {
  aspect-ratio: 1;
  background-color: #e0e0e0;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  cursor: pointer;
  transition: background-color 0.2s;
}

.grid-cell:hover {
  background-color: #d0d0d0;
}

.grid-cell.revealed {
  background-color: #ffffff;
}

.grid-cell.mine {
  background-color: #ffebee;
}

.number-1 { color: #2196f3; }
.number-2 { color: #4caf50; }
.number-3 { color: #f44336; }
.number-4 { color: #9c27b0; }
.number-5 { color: #ff9800; }
.number-6 { color: #00bcd4; }
.number-7 { color: #000000; }
.number-8 { color: #607d8b; }

.game-controls {
  display: flex;
  justify-content: center;
  margin-top: 20px;
}
</style> 