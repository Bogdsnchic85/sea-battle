<!-- Расстановка кораблей -->

<template>
  <div class="ship-placement">
    <h1>🚢 Расстановка кораблей: {{ player.name }}</h1>
    
    <div class="placement-container">
    <!-- Блок с доступными кораблями -->
      <div class="ship-list-container">
        <ShipList 
          :ships="player.ships" 
          @select-ship="handleShipSelect"
        />
        <button 
          @click="confirmPlacement" 
          :disabled="!allShipsPlaced"
          class="confirm-button"
        >
          ✏️ {{ isLastPlayer ? 'Начать битву!' : 'Готово' }}
        </button>
      </div>
      
      <!-- Игровое поле для расстановки -->
      <GameBoard 
        :board="player.board"
        :selectedShip="selectedShip"
        :shipOrientation="shipOrientation"
        @cell-click="placeShip"
      />
    </div>
    
    <!-- Инструкция -->
    <div class="instructions">
      <p>📌 Выбери корабль и кликни на поле</p>
      <p>🔄 Нажми ПРОБЕЛ для поворота</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue';

const props = defineProps({
  player: Object,
  players: Array
});

const emit = defineEmits(['ships-placed']);

const selectedShip = ref(null);
const shipOrientation = ref('horizontal');

const allShipsPlaced = computed(() => {
  return props.player.ships.every(ship => ship.placed === ship.count);
});

const isLastPlayer = computed(() => {
  return props.players.indexOf(props.player) === props.players.length - 1;
});

const handleShipSelect = (ship) => {
  selectedShip.value = ship;
};

const placeShip = (row, col) => {
  if (!selectedShip.value) {
    console.log('Корабль не выбран');
    return;
  }
  
  const shipSize = selectedShip.value.size;
  const orientation = shipOrientation.value;
  
  if (canPlaceShip(row, col, shipSize, orientation)) {
    // Размещаем корабль на доске
    for (let i = 0; i < shipSize; i++) {
      const r = orientation === 'horizontal' ? row : row + i;
      const c = orientation === 'horizontal' ? col + i : col;
      props.player.board[r][c] = 1; // 1 означает корабль
    }
    
    // Обновляем количество размещённых кораблей
    const shipIndex = props.player.ships.findIndex(
      s => s.size === selectedShip.value.size
    );
    props.player.ships[shipIndex].placed++;
    
    selectedShip.value = null;
    console.log('Корабль размещён');
  } else {
    console.log('Невозможно разместить корабль здесь');
  }
};

const canPlaceShip = (row, col, size, orientation) => {
  // Проверяем границы доски
  if (orientation === 'horizontal') {
    if (col + size > 10) return false;
  } else {
    if (row + size > 10) return false;
  }
  
  // Проверяем, что клетки свободны и вокруг них нет других кораблей
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

const confirmPlacement = () => {
  emit('ships-placed');
};

const handleKeyDown = (e) => {
  if (e.code === 'Space') {
    e.preventDefault();
    shipOrientation.value = shipOrientation.value === 'horizontal' 
      ? 'vertical' 
      : 'horizontal';
    console.log('Ориентация изменена на:', shipOrientation.value);
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

.ship-list-container {
  width: 250px;
  border-right: 2px dashed #ddd;
  padding-right: 20px;
}

.confirm-button {
  margin-top: 20px;
  padding: 12px 24px;
  background: white;
  border: 2px solid #333;
  border-radius: 5px;
  cursor: pointer;
  font-family: 'Schoolbell', cursive;
  font-size: 1.1em;
  transition: all 0.2s;
}

.confirm-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.confirm-button:not(:disabled):hover {
  background: #333;
  color: white;
}

.instructions {
  margin-top: 30px;
  padding: 15px;
  background: #f8f8f8;
  border-radius: 5px;
  color: #666;
}
</style>