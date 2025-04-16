<!-- Игровые поля -->

<template>
  <div class="player-boards">
    <!-- Собственное поле игрока -->
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
            <span v-if="opponent.hits && opponent.hits[rowIndex][colIndex]" class="miss-marker">•</span>
          </div>
        </div>
      </div>
    </div>
    
    <!-- Поле противника -->
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
            <span v-if="player.hits && player.hits[rowIndex][colIndex]" class="hit-marker">✖</span>
            <span v-if="player.misses && player.misses[rowIndex][colIndex]" class="miss-marker">•</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  player: Object,
  opponent: Object
});

const emit = defineEmits(['cell-click']);
// Функции
const getShipStyle = (row, col) => {
  if (props.player.board[row][col] === 1) {
    return { 
      backgroundColor: props.player.color,
      opacity: 0.7
    };
  }
  return {};
};
// Классы клеток
const getOwnCellClass = (cell, row, col) => {
  return {
    'ship': cell === 1,
    'hit': cell === 2,
    'miss': props.opponent.hits && props.opponent.hits[row][col]
  };
};

const getAttackCellClass = (row, col) => {
  return {
    'hit': props.player.hits && props.player.hits[row][col],
    'miss': props.player.misses && props.player.misses[row][col],
    'forbidden': props.opponent.board && props.opponent.board[row][col] === 3
  };
};

const handleCellClick = (row, col) => {
  if ((props.player.hits && props.player.hits[row][col]) || 
      (props.player.misses && props.player.misses[row][col]) || 
      (props.opponent.board && props.opponent.board[row][col] === 3)) {
    return;
  }
  emit('cell-click', row, col);
};
</script>

<style scoped>
.player-boards {
  display: flex;
  gap: 30px;
  justify-content: center;
  padding: 20px;
  background: white;
  border-radius: 5px;
  border: 1px solid #d3d3d3;
  min-height: 400px;
}

.board-container {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.game-board {
  display: inline-block;
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
  border: 1px solid v-bind(player.color);
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

h3 {
  margin: 0 0 10px 0;
  padding: 5px 10px;
  background: #f0f0f0;
  border-radius: 4px;
}

@media (max-width: 768px) {
  .player-boards {
    flex-direction: column;
  }
}
</style>