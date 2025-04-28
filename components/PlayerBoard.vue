<template>
  <div class="player-boards">
    <Transition name="slide-fade">
      <div v-if="notification.show" class="notification" :class="notification.type">
        <div class="notification-content">
          <span class="notification-icon">{{ notification.icon }}</span>
          <span class="notification-text">{{ notification.message }}</span>
        </div>
      </div>
    </Transition>

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
            :class="{
              'ship': cell === 1,
              'hit': cell === 2,
              'miss': opponent.hits?.[rowIndex]?.[colIndex]
            }"
            :style="{ backgroundColor: cell === 1 ? player.color : '' }"
          >
            <span v-if="cell === 2">✖</span>
            <span v-if="opponent.hits?.[rowIndex]?.[colIndex]">•</span>
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
            :class="{
              'hit': player.hits?.[rowIndex]?.[colIndex],
              'miss': player.misses?.[rowIndex]?.[colIndex]
            }"
            @click="handleCellClick(rowIndex, colIndex)"
          >
            <span v-if="player.hits?.[rowIndex]?.[colIndex]">✖</span>
            <span v-if="player.misses?.[rowIndex]?.[colIndex]">•</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive } from 'vue';

const props = defineProps({
  player: Object,
  opponent: Object
});

const emit = defineEmits(['cell-click']);

const notification = reactive({
  show: false,
  message: '',
  type: 'info',
  icon: 'ℹ️'
});

const showNotification = (message, type = 'info') => {
  notification.message = message;
  notification.type = type;
  
  switch(type) {
    case 'hit':
      notification.icon = '🔥';
      break;
    case 'sunk':
      notification.icon = '💥';
      break;
    case 'miss':
      notification.icon = '💦';
      break;
    case 'warning':
      notification.icon = '⚠️';
      break;
    default:
      notification.icon = 'ℹ️';
  }
  
  notification.show = true;
  setTimeout(() => {
    notification.show = false;
  }, 2000);
};

const handleCellClick = (row, col) => {
  emit('cell-click', row, col);
};

defineExpose({ showNotification });
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
  opacity: 0.7;
}

.board-cell.hit {
  background-color: rgba(255, 0, 0, 0.2);
}

.board-cell.miss {
  background-color: rgba(255, 255, 255, 0.8);
}

.notification {
  position: fixed;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  padding: 12px 24px;
  border-radius: 8px;
  color: white;
  z-index: 1000;
  box-shadow: 0 4px 12px rgba(0,0,0,0.15);
  display: flex;
  align-items: center;
  gap: 10px;
  font-family: 'Schoolbell', cursive;
}

.notification.hit {
  background: #FF6B6B;
  border-left: 5px solid #FF5252;
}

.notification.sunk {
  background: #FF9F43;
  border-left: 5px solid #FF7F26;
}

.notification.miss {
  background: #48DBFB;
  border-left: 5px solid #17C3F3;
}

.notification.warning {
  background: #FFD93D;
  border-left: 5px solid #FFC800;
  color: #333;
}

.slide-fade-enter-active {
  transition: all 0.3s ease;
}

.slide-fade-leave-active {
  transition: all 0.3s cubic-bezier(1, 0.5, 0.8, 1);
}

.slide-fade-enter-from,
.slide-fade-leave-to {
  transform: translateX(-50%) translateY(-20px);
  opacity: 0;
}

@media (max-width: 768px) {
  .player-boards {
    flex-direction: column;
  }
}
</style>