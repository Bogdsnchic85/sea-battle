<template>
  <div class="container">
    <GameSetup 
      v-if="gamePhase === 'setup'" 
      @game-start="handleGameStart" 
    />
    
    <ShipPlacement
      v-else-if="gamePhase === 'placement'"
      :player="currentPlayer"
      :players="players"
      @ships-placed="handleShipsPlaced"
    />
    
    <div v-else-if="gamePhase === 'battle'" class="battle-phase">
      <Transition name="fade">
        <TurnPopup
          v-if="showTurnPopup"
          :player="currentPlayer"
          @start-turn="showTurnPopup = false"
        />
      </Transition>
      
      <Transition name="fade">
        <GameOverPopup
          v-if="showGameOver"
          :winner="currentPlayer"
          @restart="restartGame"
        />
      </Transition>
      
      <div class="battle-container">
        <ShipTracker 
          :player="currentPlayer"
          :opponent="opponent"
          class="ship-tracker"
        />
        
        <PlayerBoard 
          ref="playerBoard"
          :player="currentPlayer" 
          :opponent="opponent"
          @cell-click="handleAttack"
        />
      </div>
    </div>

    <Transition name="fade">
      <PlacementStartPopup
        v-if="showPlacementStart"
        :player="currentPlayer"
        @start-placement="showPlacementStart = false"
      />
    </Transition>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import GameSetup from '@/components/GameSetup.vue';
import ShipPlacement from '@/components/ShipPlacement.vue';
import PlayerBoard from '@/components/PlayerBoard.vue';
import ShipTracker from '@/components/ShipTracker.vue';
import TurnPopup from '@/components/TurnPopup.vue';
import GameOverPopup from '@/components/GameOverPopup.vue';
import PlacementStartPopup from '@/components/PlacementStartPopup.vue';

const gamePhase = ref('setup');
const players = ref([]);
const currentPlayerIndex = ref(0);
const showTurnPopup = ref(false);
const showGameOver = ref(false);
const showPlacementStart = ref(false);
const playerBoard = ref(null);

const currentPlayer = computed(() => players.value[currentPlayerIndex.value] || {});
const opponent = computed(() => players.value[(currentPlayerIndex.value + 1) % 2] || {});

const initializeBoard = () => Array(10).fill().map(() => Array(10).fill(0));

const initializeShips = () => [
  { size: 4, count: 1, placed: 0 },
  { size: 3, count: 2, placed: 0 },
  { size: 2, count: 3, placed: 0 },
  { size: 1, count: 4, placed: 0 }
];

const handleGameStart = (gamePlayers) => {
  players.value = gamePlayers.map(player => ({
    ...player,
    ships: initializeShips(),
    board: initializeBoard(),
    hits: initializeBoard(),
    misses: initializeBoard(),
    sunkShips: []
  }));
  gamePhase.value = 'placement';
  showPlacementStart.value = true;
};

const handleShipsPlaced = () => {
  if (currentPlayerIndex.value < players.value.length - 1) {
    currentPlayerIndex.value++;
    showPlacementStart.value = true;
  } else {
    startBattlePhase();
  }
};

const startBattlePhase = () => {
  gamePhase.value = 'battle';
  currentPlayerIndex.value = 0;
  showTurnPopup.value = true;
};

const handleAttack = (row, col) => {
  if (showTurnPopup.value || showGameOver.value) return;
  
  const opp = opponent.value;
  const current = currentPlayer.value;
  
  if (!opp.board || !current.hits || !current.misses) return;
  
  if (current.hits[row][col] || current.misses[row][col]) {
    playerBoard.value.showNotification('Сюда уже стреляли!', 'warning');
    return;
  }
  
  if (opp.board[row][col] === 1) {
    current.hits[row][col] = 1;
    opp.board[row][col] = 2;
    
    const shipCells = findShipCells(opp.board, row, col);
    const isSunk = shipCells.every(([r, c]) => opp.board[r][c] === 2);
    
    if (isSunk) {
      current.sunkShips.push(shipCells);
      markAroundShip(current, shipCells);
      playerBoard.value.showNotification(`Уничтожен ${shipCells.length}-палубный корабль!`, 'sunk');
    } else {
      playerBoard.value.showNotification('Попадание!', 'hit');
    }
    
    if (checkGameOver()) {
      showGameOver.value = true;
    }
  } else {
    current.misses[row][col] = 1;
    playerBoard.value.showNotification('Промах!', 'miss');
    endTurn();
  }
};

const endTurn = () => {
  currentPlayerIndex.value = (currentPlayerIndex.value + 1) % 2;
  showTurnPopup.value = true;
};

const restartGame = () => {
  gamePhase.value = 'setup';
  players.value = [];
  currentPlayerIndex.value = 0;
  showGameOver.value = false;
  showTurnPopup.value = false;
  showPlacementStart.value = false;
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

const markAroundShip = (player, shipCells) => {
  const forbidden = new Set();
  
  for (const [r, c] of shipCells) {
    for (let dr = -1; dr <= 1; dr++) {
      for (let dc = -1; dc <= 1; dc++) {
        const nr = r + dr;
        const nc = c + dc;
        if (nr >= 0 && nr < 10 && nc >= 0 && nc < 10) {
          forbidden.add(`${nr},${nc}`);
        }
      }
    }
  }
  
  forbidden.forEach(key => {
    const [r, c] = key.split(',').map(Number);
    if (!player.hits[r][c] && !player.misses[r][c]) {
      player.misses[r][c] = 1;
    }
  });
};

const checkGameOver = () => {
  const opp = opponent.value;
  return opp.board?.flat().every(cell => cell !== 1) ?? false;
};
</script>

<style scoped>
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  background-color: #f9f9f5;
  font-family: 'Schoolbell', cursive;
  min-height: 100vh;
}

.battle-phase {
  display: flex;
  flex-direction: column;
  align-items: center;
  background-color: #f9f9f5;
  padding: 20px;
  border-radius: 5px;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.battle-container {
  display: flex;
  gap: 30px;
  background: white;
  padding: 20px;
  border-radius: 5px;
  border: 1px solid #d3d3d3;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}

@media (max-width: 768px) {
  .battle-container {
    flex-direction: column;
  }
  
  .ship-tracker {
    width: 100%;
  }
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>