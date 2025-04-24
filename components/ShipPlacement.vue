<template>
  <div class="ship-placement">
    <h1>🚢 Расстановка кораблей: {{ player.name }}</h1>
    
    <div v-if="showCustomAlert" class="custom-alert" :class="alertType">
      {{ alertMessage }}
    </div>
    
    <div class="placement-container">
      <div class="ship-list-container">
        <h3>Доступные корабли:</h3>
        <div 
          v-for="ship in player.ships" 
          :key="ship.size" 
          class="ship-item"
          :class="{ 
            'selected': selectedShip?.size === ship.size,
            'disabled': ship.placed >= ship.count 
          }"
          @click="selectShip(ship)"
        >
          {{ ship.size }}-палубный ({{ ship.count - ship.placed }} из {{ ship.count }})
          <div class="ship-preview" :style="previewStyle(ship)"></div>
        </div>
      </div>
      
      <div class="board-with-coordinates">
        <div class="coordinates-row">
          <div class="coordinate-cell spacer"></div>
          <div 
            v-for="(letter, index) in letters" 
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
const props = defineProps({
  player: {
    type: Object,
    required: true,
    default: () => ({
      name: 'Игрок',
      ships: [
        { size: 4, count: 1, placed: 0 },
        { size: 3, count: 2, placed: 0 },
        { size: 2, count: 3, placed: 0 },
        { size: 1, count: 4, placed: 0 }
      ],
      board: Array(10).fill().map(() => Array(10).fill(0))
    })
  },
  players: {
    type: Array,
    default: () => []
  }
});

const emit = defineEmits(['ships-placed']);

const letters = ['А', 'Б', 'В', 'Г', 'Д', 'Е', 'Ж', 'З', 'И', 'К'];
const selectedShip = ref(null);
const shipOrientation = ref('horizontal');
const hoverPosition = ref({ row: -1, col: -1 });
const selectedShipCells = ref([]);
const isMovingShip = ref(false);
const showCustomAlert = ref(false);
const alertMessage = ref('');
const alertType = ref('info');

// Вычисляемые свойства
const allShipsPlaced = computed(() => {
  return props.player.ships.every(ship => ship.placed === ship.count);
});

const isLastPlayer = computed(() => {
  return props.players.indexOf(props.player) === props.players.length - 1;
});

const previewStyle = (ship) => {
  return {
    width: shipOrientation.value === 'horizontal' ? `${ship.size * 15}px` : '15px',
    height: shipOrientation.value === 'vertical' ? `${ship.size * 15}px` : '15px'
  };
};

// Показ уведомления
const showAlert = (message, type = 'info') => {
  alertMessage.value = message;
  alertType.value = type;
  showCustomAlert.value = true;
  setTimeout(() => showCustomAlert.value = false, 2000);
};

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
const selectShip = (ship) => {
  if (ship.placed < ship.count) {
    selectedShip.value = ship;
    selectedShipCells.value = [];
    isMovingShip.value = false;
  } else {
    findAndSelectShip(ship.size);
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
      showAlert('Нельзя разместить корабль здесь!', 'error');
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
    
    if (canPlaceShip(row, col, shipSize, orientation)) {
      placeShip(row, col, shipSize, orientation);
    } else {
      showAlert('Нельзя разместить корабль здесь!', 'error');
    }
  }
};

const confirmPlacement = () => {
  if (allShipsPlaced.value) {
    emit('ships-placed');
  } else {
    showAlert('Разместите все корабли!', 'error');
  }
};

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
  // Проверка на возможность размещения
  if (!canPlaceShip(row, col, size, orientation)) {
    showAlert('Нельзя разместить корабль здесь!', 'error');
    return false;
  }

  // Размещение корабля
  for (let i = 0; i < size; i++) {
    const r = orientation === 'horizontal' ? row : row + i;
    const c = orientation === 'horizontal' ? col + i : col;
    props.player.board[r][c] = 1;
  }
  
  const shipIndex = props.player.ships.findIndex(
    s => s.size === selectedShip.value.size
  );
  props.player.ships[shipIndex].placed++;
  
  // Автовыбор следующего корабля того же типа (если есть еще)
  if (props.player.ships[shipIndex].placed < props.player.ships[shipIndex].count && 
      props.player.ships[shipIndex].count > 1) {
    selectedShip.value = { ...props.player.ships[shipIndex] };
  } else {
    selectedShip.value = null;
  }
  
  showAlert(`${size}-палубный корабль размещен`, 'success');
  return true;
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
  // Проверка выхода за границы
  if (orientation === 'horizontal') {
    if (col + size > 10) return false;
  } else {
    if (row + size > 10) return false;
  }
  
  // Проверка клеток и соседних клеток
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
        // Игнорируем клетки перемещаемого корабля
        if (isMovingShip.value && isSelectedShipCell(r, c)) continue;
        
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
  font-family: 'Arial', sans-serif;
}

.custom-alert {
  position: fixed;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  padding: 12px 24px;
  border-radius: 5px;
  background: #4CAF50;
  color: white;
  z-index: 1000;
  animation: fadeInOut 2s ease-in-out;
}

.custom-alert.error {
  background: #f44336;
}

.custom-alert.info {
  background: #2196F3;
}

@keyframes fadeInOut {
  0% { opacity: 0; top: 0; }
  10% { opacity: 1; top: 20px; }
  90% { opacity: 1; top: 20px; }
  100% { opacity: 0; top: 0; }
}

.placement-container {
  display: flex;
  gap: 40px;
  margin-top: 30px;
}

.ship-list-container {
  width: 250px;
  padding: 15px;
  background: #f5f5f5;
  border-radius: 8px;
}

.ship-item {
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  cursor: pointer;
  transition: all 0.2s;
}

.ship-item.selected {
  border-color: #4CAF50;
  background-color: #e8f5e9;
}

.ship-item.disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.ship-preview {
  height: 15px;
  margin-top: 5px;
  background-color: #555;
  border-radius: 3px;
}

.board-with-coordinates {
  display: flex;
  flex-direction: column;
}

.coordinates-row {
  display: flex;
  margin-left: 30px;
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

.board-row {
  display: flex;
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