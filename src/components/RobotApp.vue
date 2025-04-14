<template>
  <div class="robot-app">
    <!-- Audio elements for sounds -->
    <audio ref="leftSound" src="/sounds/turn.m4a" preload="auto"></audio>
    <audio ref="rightSound" src="/sounds/turn.m4a" preload="auto"></audio>
    <audio ref="forwardSound" src="/sounds/run.m4a" preload="auto"></audio>
    <audio ref="strawberrySound" src="/sounds/am.m4a" preload="auto"></audio>
    
    <div class="grid-container">
      <div 
        v-for="(cell, index) in grid" 
        :key="index"
        class="grid-cell"
        :class="{ 
          'has-robot': cell.hasRobot,
          'has-strawberry': cell.hasStrawberry,
          'has-wall': cell.hasWall
        }"
      >
        <v-icon 
          v-if="cell.hasRobot" 
          size="32" 
          color="primary"
          :style="{
            transform: `rotate(${
              robotDirection === NORTH ? '0deg' :
              robotDirection === EAST ? '90deg' :
              robotDirection === SOUTH ? '180deg' :
              '270deg'
            })`
          }"
        >mdi-robot</v-icon>
        <v-icon 
          v-if="cell.hasStrawberry" 
          size="32" 
          color="red"
        >mdi-fruit-cherries</v-icon>
        <v-icon 
          v-if="cell.hasWall" 
          size="32" 
          color="grey-darken-2"
        >mdi-wall</v-icon>
      </div>
    </div>
    
    <div class="controls-panel">
      
      <div class="strawberry-counter mb-4">
        <v-icon color="red" size="24" class="mr-2">mdi-fruit-cherries</v-icon>
        <span>Frutillas recolectadas: {{ strawberryCount }}</span>
      </div>
      
      <div class="control-buttons mb-4">
        <div class="d-flex">
          <v-btn
            color="primary"
            class="flex-grow-1 mr-1"
            @click="startSimulation"
            :disabled="isRunning"
          >
            <v-icon start>mdi-play</v-icon>
            Iniciar
          </v-btn>
          
          <v-btn
            color="error"
            class="flex-grow-1 ml-1"
            @click="resetSimulation"
          >
            <v-icon start>mdi-refresh</v-icon>
            Reiniciar
          </v-btn>
        </div>
      </div>

      <div class="instructions">
        <h4 class="mb-2">Instrucciones</h4>
        
        <div class="instructions-display mb-4">
          <div 
            v-for="(instruction, index) in instructionsArray" 
            :key="index"
            class="instruction-item"
          >
            <v-icon
              v-if="instruction === LEFT"
              color="primary"
              size="24"
              class="mr-2"
            >mdi-arrow-u-left-top</v-icon>
            <v-icon
              v-else-if="instruction === RIGHT"
              color="primary"
              size="24"
              class="mr-2"
            >mdi-arrow-u-right-top</v-icon>
            <v-icon
              v-else-if="instruction === FORWARD"
              color="primary"
              size="24"
              class="mr-2"
            >mdi-arrow-up</v-icon>
            <span v-else>{{ instruction }}</span>
            <v-btn
              icon
              variant="text"
              size="small"
              color="error"
              class="ml-2"
              @click="removeInstruction(index)"
              :disabled="isRunning"
            >
              <v-icon>mdi-close</v-icon>
            </v-btn>
          </div>
          <div v-if="instructionsArray.length === 0" class="no-instructions">
            No hay instrucciones
          </div>
        </div>
        
        <div class="instruction-actions mb-4">
          <v-btn
            color="error"
            variant="outlined"
            block
            :disabled="isRunning || instructionsArray.length === 0"
            @click="clearAllInstructions"
          >
            <v-icon start>mdi-delete</v-icon>
            Borrar todas
          </v-btn>
        </div>
        
        <div class="instruction-buttons d-flex justify-center">
          <v-btn
            v-for="instruction in [LEFT, RIGHT, FORWARD]"
            :key="instruction"
            color="primary"
            variant="outlined"
            class="mx-2"
            :disabled="isRunning"
            @click="addInstruction(instruction)"
          >
            <v-icon
              v-if="instruction === LEFT"
            >mdi-arrow-u-left-top</v-icon>
            <v-icon
              v-if="instruction === RIGHT"
            >mdi-arrow-u-right-top</v-icon>
            <v-icon
              v-if="instruction === FORWARD"
            >mdi-arrow-up</v-icon>
          </v-btn>
          <v-btn
            color="primary"
            variant="outlined"
            class="mx-2"
            :disabled="isRunning || ![LEFT, RIGHT, FORWARD].includes(instructionsArray[instructionsArray.length - 1])"
            @click="showNumberDialog = true"
          >
            <v-icon>mdi-numeric</v-icon>
          </v-btn>
        </div>
      </div>
    </div>

    <!-- Dialog para ingresar número -->
    <v-dialog
      v-model="showNumberDialog"
      max-width="400"
    >
      <v-card>
        <v-card-title class="text-h5">
          Ingresar número de repeticiones
        </v-card-title>
        <v-card-text>
          <v-number-input 
          v-model="numberInput" 
          label="Número de repeticiones"
           min="2"
            max="10"
          control-variant="split">
        </v-number-input>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
            color="error"
            variant="text"
            @click="showNumberDialog = false"
          >
            Cancelar
          </v-btn>
          <v-btn
            color="primary"
            variant="text"
            @click="addNumberInstruction"
          >
            Aceptar
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'

// Add audio refs
const leftSound = ref(null)
const rightSound = ref(null)
const forwardSound = ref(null)
const strawberrySound = ref(null)

// DIRECTIONS
const NORTH = 'NORTH'
const WEST = 'WEST'
const SOUTH = 'SOUTH'
const EAST = 'EAST'
const directions = [NORTH, WEST, SOUTH, EAST]

// INSTRUCTIONS
const FORWARD = 'FORWARD'
const LEFT = 'LEFT'
const RIGHT = 'RIGHT'

const GRID_ROWS = 5
const GRID_COLS = 10
const grid = ref([])
const robotPosition = ref({ row: 0, col: 0 })
const robotDirection = ref('NORTH') 
const instructionsArray = ref([])
const isRunning = ref(false)
const strawberryPosition = ref({ row: 0, col: 0 })
const strawberryCount = ref(0)
const showNumberDialog = ref(false)
const numberInput = ref(2)

const initializeGrid = () => {
  grid.value = []
  for (let row = 0; row < GRID_ROWS; row++) {
    for (let col = 0; col < GRID_COLS; col++) {
      grid.value.push({
        row,
        col,
        hasRobot: false,
        hasStrawberry: false,
        hasWall: false
      })
    }
  }
}

const generateWalls = () => {
  // Limpiar paredes existentes
  grid.value.forEach(cell => {
    cell.hasWall = false
  })

  // Generar un número aleatorio de paredes (entre 5 y 10)
  const numWalls = Math.floor(Math.random() * 6) + 5
  
  // Función para verificar si una posición es válida para una pared
  const isValidWallPosition = (row, col) => {
    // No poner paredes en la posición del robot o la frutilla
    if ((row === robotPosition.value.row && col === robotPosition.value.col) ||
        (row === strawberryPosition.value.row && col === strawberryPosition.value.col)) {
      return false
    }
    
    // No poner paredes adyacentes que bloqueen completamente
    const adjacentCells = [
      { row: row - 1, col },
      { row: row + 1, col },
      { row, col: col - 1 },
      { row, col: col + 1 }
    ]
    
    // Verificar que no haya demasiadas paredes adyacentes
    const adjacentWalls = adjacentCells.filter(cell => 
      cell.row >= 0 && cell.row < GRID_ROWS &&
      cell.col >= 0 && cell.col < GRID_COLS &&
      grid.value.find(c => c.row === cell.row && c.col === cell.col)?.hasWall
    ).length
    
    return adjacentWalls < 3
  }

  // Colocar paredes aleatoriamente
  let wallsPlaced = 0
  while (wallsPlaced < numWalls) {
    const randomRow = Math.floor(Math.random() * GRID_ROWS)
    const randomCol = Math.floor(Math.random() * GRID_COLS)
    
    if (isValidWallPosition(randomRow, randomCol)) {
      const cell = grid.value.find(c => c.row === randomRow && c.col === randomCol)
      if (cell) {
        cell.hasWall = true
        wallsPlaced++
      }
    }
  }
}

const placeRobotRandomly = () => {
  const randomRow = Math.floor(Math.random() * GRID_ROWS)
  const randomCol = Math.floor(Math.random() * GRID_COLS)
  robotPosition.value = { row: randomRow, col: randomCol }
  updateRobotPosition()
}

const placeStrawberryRandomly = () => {
  let newPosition
  do {
    newPosition = {
      row: Math.floor(Math.random() * GRID_ROWS),
      col: Math.floor(Math.random() * GRID_COLS)
    }
  } while (
    (newPosition.row === robotPosition.value.row && 
     newPosition.col === robotPosition.value.col)
  )
  strawberryPosition.value = newPosition
  updateStrawberryPosition()
}

const updateRobotPosition = () => {
  grid.value.forEach(cell => {
    cell.hasRobot = cell.row === robotPosition.value.row && cell.col === robotPosition.value.col
  })
}

const updateStrawberryPosition = () => {
  grid.value.forEach(cell => {
    cell.hasStrawberry = cell.row === strawberryPosition.value.row && cell.col === strawberryPosition.value.col
  })
}

const rotateLeft = () => {
  const currentIndex = directions.indexOf(robotDirection.value)
  robotDirection.value = directions[(currentIndex + 1) % 4]
  leftSound.value.play()
}

const rotateRight = () => {
  const currentIndex = directions.indexOf(robotDirection.value)
  robotDirection.value = directions[(currentIndex + 1) % 4]
  rightSound.value.play()
}

const moveForward = () => {
  let newPosition = { ...robotPosition.value }
  
  switch (robotDirection.value) {
    case NORTH:
      newPosition.row--
      break
    case SOUTH:
      newPosition.row++
      break
    case EAST:
      newPosition.col++
      break
    case WEST:
      newPosition.col--
      break
  }

  if (newPosition.row >= 0 && newPosition.row < GRID_ROWS &&
      newPosition.col >= 0 && newPosition.col < GRID_COLS) {
    const targetCell = grid.value.find(cell => 
      cell.row === newPosition.row && cell.col === newPosition.col
    )
    
    if (targetCell && !targetCell.hasWall) {
      robotPosition.value = newPosition
      forwardSound.value.play()
    }
  }
}


const startSimulation = async () => {
  if (instructionsArray.value.length === 0) return
  
  isRunning.value = true
  const processedSteps = []
  
  for (let i = 0; i < instructionsArray.value.length; i++) {
    const step = instructionsArray.value[i]
    if (!isNaN(step)) {
      const repeatCount = parseInt(step)
      const previousInstruction = processedSteps[processedSteps.length - 1]
      for (let j = 1; j < repeatCount; j++) {
        processedSteps.push(previousInstruction)
      }
    } else {
      processedSteps.push(step)
    }
  }
  
  for (const step of processedSteps) {
    if (!isRunning.value) break
    
    switch (step) {
      case LEFT:
        rotateLeft()
        break
      case RIGHT:
        rotateRight()
        break
      case FORWARD:
        moveForward() // TODO ver si al chocar frena la simulacion
        break
    }
    
    updateRobotPosition()
    
    const hasCollision = robotPosition.value.row === strawberryPosition.value.row && 
                        robotPosition.value.col === strawberryPosition.value.col
    
    if (hasCollision) {
      strawberryCount.value++

      initializeGrid()
      updateRobotPosition()
      placeStrawberryRandomly()
      generateWalls()

      strawberrySound.value.play()
      isRunning.value = false
      instructionsArray.value = []
      break
    }
    
    await new Promise(resolve => setTimeout(resolve, 500))
  }
  
  isRunning.value = false
  instructionsArray.value = []
}

const resetSimulation = () => {
  isRunning.value = false
  instructionsArray.value = []
  robotDirection.value = NORTH
  strawberryCount.value = 0
  initializeGrid()
  placeRobotRandomly()
  placeStrawberryRandomly()
  generateWalls()
}

const addInstruction = (instruction) => {
  instructionsArray.value.push(instruction)
}

const addNumberInstruction = () => {
  if (instructionsArray.value.length > 0 && numberInput.value >= 2) {
    instructionsArray.value.push(numberInput.value.toString())
    showNumberDialog.value = false
    numberInput.value = 2 // Resetear el valor
  }
}

const removeInstruction = (index) => {
  instructionsArray.value.splice(index, 1)
}

const clearAllInstructions = () => {
  instructionsArray.value = []
}

onMounted(() => {
  initializeGrid()
  placeRobotRandomly()
  placeStrawberryRandomly()
  generateWalls()
})
</script>

<style scoped>
.robot-app {
  display: flex;
  gap: 20px;
  padding: 20px;
  height: 100%;
}

.grid-container {
  display: grid;
  grid-template-columns: repeat(10, 70px);
  grid-template-rows: repeat(5, 70px);
  gap: 1px;
  background-color: #f0f0f0;
  padding: 10px;
  border-radius: 8px;
  width: fit-content;
  height: fit-content;
}

.grid-cell {
  width: 70px;
  height: 70px;
  background-color: white;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.has-robot {
  background-color: #e3f2fd;
}

.has-strawberry {
  background-color: #ffebee;
}

.has-wall {
  background-color: #9e9e9e;
}

.controls-panel {
  flex: 1;
  background-color: #1e1e1e;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  color: white;
}

.instructions {
  margin-top: 20px;
}

.instructions-display {
  min-height: 300px;
  background-color: rgba(255, 255, 255, 0.1);
  border-radius: 4px;
  padding: 8px;
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  align-content: flex-start;
  align-items: flex-start;
}

.instruction-item {
  display: flex;
  align-items: center;
  background-color: rgba(255, 255, 255, 0.2);
  padding: 4px 8px;
  border-radius: 4px;
  transition: background-color 0.2s;
  height: fit-content;
}

.instruction-item:hover {
  background-color: rgba(255, 255, 255, 0.3);
}

.no-instructions {
  color: rgba(255, 255, 255, 0.5);
  font-style: italic;
  width: 100%;
  text-align: center;
  padding: 8px;
}

.instruction-actions {
  display: flex;
  gap: 8px;
}

.instruction-buttons {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.strawberry-counter {
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.2rem;
  padding: 8px;
  background-color: rgba(255, 255, 255, 0.1);
  border-radius: 4px;
}

:deep(.v-textarea textarea) {
  color: white !important;
}

:deep(.v-textarea .v-field__input) {
  color: white !important;
}

:deep(.v-textarea .v-field__outline) {
  color: rgba(255, 255, 255, 0.5) !important;
}

:deep(.v-textarea .v-field__outline--focused) {
  color: white !important;
}

:deep(.v-label) {
  color: rgba(255, 255, 255, 0.7) !important;
}

:deep(.v-btn) {
  text-transform: none !important;
}

.log-item {
  font-family: monospace;
  font-size: 0.9rem;
  padding: 4px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.log-item:last-child {
  border-bottom: none;
}


</style> 