<!-- Расстановка кораблей -->

<template>
  <div class="ship-placement">
    <h1>🚢 Расстановка кораблей: {{ player.name }}</h1>
    
    <div class="placement-container">
      <ShipList 
        :ships="player.ships" 
        @select-ship="handleShipSelect"
      />
      
      <div class="board-with-coordinates">
        <div class="letters-row">
          <div class="coordinate-cell"></div>
          <div 
            v-for="letter in letters" 
            :key="letter" 
            class="coordinate-cell letter"
          >
            {{ letter }}
          </div>
        </div>
        
        <div 
          v-for="(row, rowIndex) in player.board" 
          :key="rowIndex" 
          class="board-row"
        >
          <div class="coordinate-cell number">
            {{ rowIndex + 1 }}
          </div>
          <div
            v-for="(cell, colIndex) in row"
            :key="colIndex"
            class="board-cell"
            :class="{
              'ship-placed': cell === 1 && !isSelectedShipCell(rowIndex, colIndex),
              'can-place': canPlaceHere(rowIndex, colIndex),
              'cannot-place': cannotPlaceHere(rowIndex, colIndex),
              'selected-ship': isSelectedShipCell(rowIndex, colIndex)
            }"
            @click="handleCellClick(rowIndex, colIndex)"
            @mouseover="hoverCell(rowIndex, colIndex)"
            @mouseleave="resetHover()"
          ></div>
        </div>
      </div>
    </div>
    
    <div class="controls">
      <button 
        @click="confirmPlacement" 
        :disabled="!allShipsPlaced"
        class="confirm-button"
      >
        {{ isLastPlayer ? 'Начать битву!' : 'Готово' }}
      </button>
      <button 
        @click="rotateShip"
        :disabled="!selectedShip"
        class="rotate-button"
      >
        Повернуть корабль (Пробел)
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue';
import ShipList from './ShipList.vue';

const props = defineProps({
  player: Object,
  players: Array
});

const emit = defineEmits(['ships-placed']);

const letters = ['А', 'Б', 'В', 'Г', 'Д', 'Е', 'Ж', 'З', 'И', 'К'];
const selectedShip = ref(null);
const shipOrientation = ref('horizontal');
const hoverPosition = ref({ row: -1, col: -1 });
const selectedShipCells = ref([]);
const isMovingShip = ref(false);

// Вычисляемые свойства
const allShipsPlaced = computed(() => {
  return props.player.ships.every(ship => ship.placed === ship.count);
});

const isLastPlayer = computed(() => {
  return props.players.indexOf(props.player) === props.players.length - 1;
});

// Проверка клеток
const isSelectedShipCell = (row, col) => {
  return selectedShipCells.value.some(([r, c]) => r === row && c === col);
};

const canPlaceHere = (row, col) => {
  if (!selectedShip.value || hoverPosition.value.row === -1) return false;
  
  const size = selectedShip.value.size;
  const orientation = shipOrientation.value;
  
  if (orientation === 'horizontal') {
    if (row !== hoverPosition.value.row) return false;
    if (col < hoverPosition.value.col || col >= hoverPosition.value.col + size) return false;
  } else {
    if (col !== hoverPosition.value.col) return false;
    if (row < hoverPosition.value.row || row >= hoverPosition.value.row + size) return false;
  }
  
  return canPlaceShip(hoverPosition.value.row, hoverPosition.value.col, size, orientation);
};

const cannotPlaceHere = (row, col) => {
  if (!selectedShip.value || hoverPosition.value.row === -1) return false;
  
  const size = selectedShip.value.size;
  const orientation = shipOrientation.value;
  
  if (orientation === 'horizontal') {
    if (row !== hoverPosition.value.row) return false;
    if (col < hoverPosition.value.col || col >= hoverPosition.value.col + size) return false;
  } else {
    if (col !== hoverPosition.value.col) return false;
    if (row < hoverPosition.value.row || row >= hoverPosition.value.row + size) return false;
  }
  
  return !canPlaceShip(hoverPosition.value.row, hoverPosition.value.col, size, orientation);
};

// Обработчики событий
const handleShipSelect = (ship) => {
  if (ship.placed >= ship.count) {
    findAndSelectShip(ship.size);
  } else {
    selectedShip.value = ship;
    selectedShipCells.value = [];
    isMovingShip.value = false;
  }
};

const rotateShip = () => {
  if (selectedShip.value) {
    shipOrientation.value = shipOrientation.value === 'horizontal' 
      ? 'vertical' 
      : 'horizontal';
  }
};

const handleCellClick = (row, col) => {
  if (!selectedShip.value) return;
  
  if (isMovingShip.value) {
    const shipSize = selectedShip.value.size;
    const orientation = shipOrientation.value;
    
    if (canPlaceShip(row, col, shipSize, orientation)) {
      placeShip(row, col, shipSize, orientation);
      isMovingShip.value = false;
      selectedShipCells.value = [];
    } else {
      returnShipToOriginalPosition();
    }
  } else if (props.player.board[row][col] === 1) {
    const shipCells = findShipCells(row, col);
    const shipSize = shipCells.length;
    selectedShip.value = props.player.ships.find(s => s.size === shipSize);
    selectedShipCells.value = shipCells;
    isMovingShip.value = true;
    removeShip(shipCells);
  } else if (selectedShip.value) {
    const shipSize = selectedShip.value.size;
    const orientation = shipOrientation.value;
    placeShip(row, col, shipSize, orientation);
  }
};

const confirmPlacement = () => {
  if (allShipsPlaced.value) {
    emit('ships-placed');
  }
};

// Вспомогательные функции
const findAndSelectShip = (size) => {
  for (let row = 0; row < 10; row++) {
    for (let col = 0; col < 10; col++) {
      if (props.player.board[row][col] === 1) {
        const shipCells = findShipCells(row, col);
        if (shipCells.length === size) {
          selectedShip.value = props.player.ships.find(s => s.size === size);
          selectedShipCells.value = shipCells;
          isMovingShip.value = true;
          removeShip(shipCells);
          return;
        }
      }
    }
  }
};

const removeShip = (cells) => {
  cells.forEach(([row, col]) => {
    props.player.board[row][col] = 0;
  });
  const shipIndex = props.player.ships.findIndex(
    s => s.size === selectedShip.value.size
  );
  props.player.ships[shipIndex].placed--;
};

const placeShip = (row, col, size, orientation) => {
  for (let i = 0; i < size; i++) {
    const r = orientation === 'horizontal' ? row : row + i;
    const c = orientation === 'horizontal' ? col + i : col;
    props.player.board[r][c] = 1;
  }
  
  const shipIndex = props.player.ships.findIndex(
    s => s.size === selectedShip.value.size
  );
  props.player.ships[shipIndex].placed++;
  
  selectedShip.value = null;
};

const findShipCells = (startRow, startCol) => {
  const directions = [[0,1],[1,0],[0,-1],[-1,0]];
  const visited = new Set();
  const queue = [[startRow, startCol]];
  const shipCells = [];
  
  while (queue.length > 0) {
    const [r, c] = queue.shift();
    const key = `${r},${c}`;
    
    if (visited.has(key) || props.player.board[r][c] !== 1) continue;
    visited.add(key);
    shipCells.push([r, c]);
    
    for (const [dr, dc] of directions) {
      const nr = r + dr;
      const nc = c + dc;
      if (nr >= 0 && nr < 10 && nc >= 0 && nc < 10) {
        queue.push([nr, nc]);
      }
    }
  }
  
  return shipCells;
};

const canPlaceShip = (row, col, size, orientation) => {
  if (orientation === 'horizontal') {
    if (col + size > 10) return false;
  } else {
    if (row + size > 10) return false;
  }
  
  for (let i = -1; i <= size; i++) {
    for (let j = -1; j <= 1; j++) {
      let r, c;
      if (orientation === 'horizontal') {
        r = row + j;
        c = col + i;
      } else {
        r = row + i;
        c = col + j;
      }
      
      if (r >= 0 && r < 10 && c >= 0 && c < 10) {
        if (props.player.board[r][c] !== 0) {
          return false;
        }
      }
    }
  }
  
  return true;
};

const returnShipToOriginalPosition = () => {
  selectedShipCells.value.forEach(([row, col]) => {
    props.player.board[row][col] = 1;
  });
  const shipIndex = props.player.ships.findIndex(
    s => s.size === selectedShip.value.size
  );
  props.player.ships[shipIndex].placed++;
  isMovingShip.value = false;
  selectedShipCells.value = [];
};

const hoverCell = (row, col) => {
  if (selectedShip.value) {
    hoverPosition.value = { row, col };
  }
};

const resetHover = () => {
  hoverPosition.value = { row: -1, col: -1 };
};

// Обработка нажатия клавиш
const handleKeyDown = (e) => {
  if (e.code === 'Space') {
    e.preventDefault();
    rotateShip();
  }
};

onMounted(() => {
  window.addEventListener('keydown', handleKeyDown);
});

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeyDown);
});
</script>

<style scoped>
.ship-placement {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  background: white;
  border-radius: 5px;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
  font-family: 'Schoolbell', cursive;
}

.placement-container {
  display: flex;
  gap: 40px;
  margin-top: 30px;
}

.board-with-coordinates {
  display: flex;
  flex-direction: column;
}

.letters-row {
  display: flex;
  margin-left: 30px;
}

.board-row {
  display: flex;
}

.coordinate-cell {
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.9em;
}

.letter, .number {
  font-weight: bold;
  color: #333;
}

.board-cell {
  width: 30px;
  height: 30px;
  border: 1px solid #ccc;
  background-color: #f0f8ff;
  cursor: pointer;
  transition: all 0.2s;
}

.board-cell.can-place {
  background-color: #c8e6c9;
  border-color: #4caf50;
}

.board-cell.cannot-place {
  background-color: #ffcdd2;
  border-color: #f44336;
}

.board-cell.ship-placed {
  background-color: #bbdefb;
  border-color: #2196f3;
}

.board-cell.selected-ship {
  background-color: #ffecb3;
  border-color: #ffa000;
}

.controls {
  display: flex;
  gap: 20px;
  justify-content: center;
  margin-top: 30px;
}

.confirm-button, .rotate-button {
  padding: 12px 24px;
  border-radius: 5px;
  cursor: pointer;
  font-family: 'Schoolbell', cursive;
  font-size: 1.1em;
  transition: all 0.2s;
  border: none;
}

.confirm-button {
  background: #4CAF50;
  color: white;
}

.confirm-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.confirm-button:not(:disabled):hover {
  background: #388E3C;
}

.rotate-button {
  background: #FFC107;
  color: #333;
}

.rotate-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.rotate-button:not(:disabled):hover {
  background: #FFA000;
}

@media (max-width: 768px) {
  .placement-container {
    flex-direction: column;
  }
  
  .controls {
    flex-direction: column;
  }
}
</style>