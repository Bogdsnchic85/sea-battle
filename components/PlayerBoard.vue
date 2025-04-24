<template>
  <div class="player-boards">
    <!-- Уведомление -->
    <div v-if="notification.show" class="notification" :class="notification.type">
      {{ notification.message }}
    </div>
    
    <div class="board-container own-board">
      <h3>🛡️ {{ player.name }}</h3>
      <div class="game-board">
        <div 
          v-for="(row, rowIndex) in player.board" 
          :key="'own-row-' + rowIndex" 
          class="board-row"
        >
          <div
            v-for="(cell, colIndex) in row"
            :key="'own-cell-' + rowIndex + '-' + colIndex"
            class="board-cell"
            :class="getOwnCellClass(cell, rowIndex, colIndex)"
            :style="getShipStyle(rowIndex, colIndex)"
          >
            <span v-if="cell === 2" class="hit-marker">✖</span>
            <span v-if="opponent.hits?.[rowIndex]?.[colIndex]" class="miss-marker">•</span>
          </div>
        </div>
      </div>
    </div>
    
    <div class="board-container enemy-board">
      <h3>🎯 {{ opponent.name }}</h3>
      <div class="game-board">
        <div 
          v-for="(row, rowIndex) in opponent.board" 
          :key="'enemy-row-' + rowIndex" 
          class="board-row"
        >
          <div
            v-for="(cell, colIndex) in row"
            :key="'enemy-cell-' + rowIndex + '-' + colIndex"
            class="board-cell"
            :class="getAttackCellClass(rowIndex, colIndex)"
            @click="handleCellClick(rowIndex, colIndex)"
          >
            <span v-if="player.hits?.[rowIndex]?.[colIndex]" class="hit-marker">✖</span>
            <span v-if="player.misses?.[rowIndex]?.[colIndex]" class="miss-marker">•</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed } from 'vue';

const props = defineProps({
  player: Object,
  opponent: Object
});

const emit = defineEmits(['cell-click']);

const notification = reactive({
  show: false,
  message: '',
  type: 'info' // 'info' | 'success' | 'error'
});

const showNotification = (message, type = 'info') => {
  notification.message = message;
  notification.type = type;
  notification.show = true;
  setTimeout(() => notification.show = false, 3000);
};

const getShipStyle = (row, col) => {
  if (props.player.board[row][col] === 1) {
    return { 
      backgroundColor: props.player.color,
      opacity: 0.7
    };
  }
  return {};
};

const getOwnCellClass = (cell, row, col) => {
  return {
    'ship': cell === 1,
    'hit': cell === 2,
    'miss': props.opponent.hits?.[row]?.[col]
  };
};

const getAttackCellClass = (row, col) => {
  return {
    'hit': props.player.hits?.[row]?.[col],
    'miss': props.player.misses?.[row]?.[col],
    'forbidden': props.opponent.board?.[row]?.[col] === 3
  };
};

const handleCellClick = (row, col) => {
  if (
    props.player.hits?.[row]?.[col] || 
    props.player.misses?.[row]?.[col] || 
    props.opponent.board?.[row]?.[col] === 3
  ) {
    showNotification('Сюда уже стреляли!', 'error');
    return;
  }
  
  emit('cell-click', row, col);
  
  // Показываем уведомление о результате выстрела
  if (props.opponent.board?.[row]?.[col] === 1) {
    const shipCells = findShipCells(props.opponent.board, row, col);
    const isSunk = shipCells.every(([r, c]) => props.opponent.board[r][c] === 2);
    
    if (isSunk) {
      showNotification(`Уничтожен ${shipCells.length}-палубный корабль!`, 'success');
    } else {
      showNotification('Попадание!', 'success');
    }
  } else {
    showNotification('Промах!', 'error');
  }
};

const findShipCells = (board, startRow, startCol) => {
  const directions = [[0,1],[1,0],[0,-1],[-1,0]];
  const visited = new Set();
  const queue = [[startRow, startCol]];
  const shipCells = [];
  
  while (queue.length > 0) {
    const [r, c] = queue.shift();
    const key = `${r},${c}`;
    
    if (visited.has(key) || board[r][c] === 0) continue;
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
</script>

<style scoped>
.player-boards {
  display: flex;
  gap: 30px;
  justify-content: center;
  position: relative;
}

.board-container {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.game-board {
  border: 2px solid #333;
  background: #f8f8f8;
  margin-top: 10px;
}

.board-row {
  display: flex;
}

.board-cell {
  width: 30px;
  height: 30px;
  border: 1px solid #aaa;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  background-color: rgba(173, 216, 230, 0.3);
}

.board-cell.ship {
  background-color: v-bind('player.color + "80"');
  border: 1px solid v-bind('player.color');
}

.board-cell.hit {
  background-color: rgba(255, 0, 0, 0.2);
}

.board-cell.miss {
  background-color: rgba(255, 255, 255, 0.8);
}

.hit-marker {
  color: #c00;
  font-size: 20px;
  font-weight: bold;
}

.miss-marker {
  color: #333;
  font-size: 20px;
}

.board-cell.forbidden {
  background-color: #f0f0f0;
  cursor: not-allowed;
}

.notification {
  position: absolute;
  top: -40px;
  right: 0;
  padding: 10px 15px;
  border-radius: 5px;
  color: white;
  z-index: 100;
  animation: slideIn 0.3s ease-out;
}

.notification.success {
  background-color: #4CAF50;
}

.notification.error {
  background-color: #F44336;
}

@keyframes slideIn {
  from {
    transform: translateY(-20px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

@media (max-width: 768px) {
  .player-boards {
    flex-direction: column;
  }
}
</style>