<template>
  <div class="container">
    <!-- Фаза настройки игры  -->
    <GameSetup 
      v-if="gamePhase === 'setup'" 
      @game-start="handleGameStart" 
    />
    
    <!-- Фаза расстановки кораблей на поле -->
    <ShipPlacement
      v-else-if="gamePhase === 'placement'"
      :player="currentPlayer"
      :players="players"
      @ships-placed="handleShipsPlaced"
    />
    
    <!-- Фаза боя -->
    <div v-else-if="gamePhase === 'battle'" class="battle-phase">
      <!-- Попап, уведомляющий о смене хода -->
      <TurnPopup
        v-if="showTurnPopup"
        :player="currentPlayer"
        @start-turn="showTurnPopup = false"
      />
      
      <div class="battle-container">
        <ShipTracker 
          :player="currentPlayer"
          :opponent="opponent"
          class="ship-tracker"
        />
        
        <!-- Игровые полея -->
        <div class="battle-boards">
          <PlayerBoard 
            :player="currentPlayer" 
            :opponent="opponent"
            @cell-click="handleAttack"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  data() {
    return {
      gamePhase: 'setup', 
      players: [],        
      currentPlayerIndex: 0, 
      showTurnPopup: false  
    }
  },
  computed: {
    currentPlayer() {
      return this.players[this.currentPlayerIndex]
    },
    opponent() {
      return this.players[(this.currentPlayerIndex + 1) % 2]
    }
  },
  methods: {
    // Обработчик начала игры 
    handleGameStart(gamePlayers) {
      this.players = gamePlayers.map(player => ({
        ...player,
        ships: this.initializeShips(),  
        board: this.initializeBoard(),   
        hits: this.initializeBoard(),    
        misses: this.initializeBoard(),  
        sunkShips: []                    
      }))
      this.gamePhase = 'placement' 
    },
    
    // Обработчик завершения расстановки кораблей
    handleShipsPlaced() {
      if (this.currentPlayerIndex < this.players.length - 1) {
        this.currentPlayerIndex++
      } else {
        this.gamePhase = 'battle'
        this.currentPlayerIndex = 0
        this.showTurnPopup = true 
      }
    },
    
    // Обработчик атаки 
    handleAttack(row, col) {
      if (this.showTurnPopup) return 
      
      const opp = this.opponent
      const current = this.currentPlayer
      
      if (current.hits[row][col] || current.misses[row][col]) {
        alert('Вы уже стреляли в эту клетку!')
        return
      }
      
      // Проверка попадания
      if (opp.board[row][col] === 1) {
        // Попадание - отмечаем на полях
        current.hits[row][col] = 1
        opp.board[row][col] = 2
        
        // Поиск всех клеток корабля
        const shipCells = this.findShipCells(opp.board, row, col)
        // Проверка, потоплен ли корабль
        const isSunk = shipCells.every(([r, c]) => opp.board[r][c] === 2)
        
        if (isSunk) {
          current.sunkShips.push(shipCells)
          this.markAroundShip(current, shipCells)
        }
        
        // Проверка условия победы
        if (this.checkGameOver()) {
          alert(`${current.name} победил!`)
          this.gamePhase = 'setup' 
          return
        }
      } else {
        current.misses[row][col] = 1
        this.endTurn() 
      }
    },
    
    findShipCells(board, startRow, startCol) {
      const directions = [[0,1],[1,0],[0,-1],[-1,0]] 
      const visited = new Set() 
      const queue = [[startRow, startCol]] 
      const shipCells = [] 
      
      while (queue.length > 0) {
        const [r, c] = queue.shift()
        const key = `${r},${c}`
        
        if (visited.has(key) || board[r][c] === 0) continue
        visited.add(key)
        shipCells.push([r, c])
        
        for (const [dr, dc] of directions) {
          const nr = r + dr
          const nc = c + dc
          if (nr >= 0 && nr < 10 && nc >= 0 && nc < 10) {
            queue.push([nr, nc])
          }
        }
      }
      
      return shipCells
    },
    
    // Отметка клеток вокруг потопленного корабля
    markAroundShip(player, shipCells) {
      const forbidden = new Set() 
      
      for (const [r, c] of shipCells) {
        for (let dr = -1; dr <= 1; dr++) {
          for (let dc = -1; dc <= 1; dc++) {
            const nr = r + dr
            const nc = c + dc
            if (nr >= 0 && nr < 10 && nc >= 0 && nc < 10) {
              forbidden.add(`${nr},${nc}`)
            }
          }
        }
      }
      
      // Отмечаем клетки вокруг как промахи
      forbidden.forEach(key => {
        const [r, c] = key.split(',').map(Number)
        if (!player.hits[r][c] && !player.misses[r][c]) {
          player.misses[r][c] = 1
        }
      })
    },
    
    // Завершение хода
    endTurn() {
      this.showTurnPopup = true // Показ попапа смены хода
      this.currentPlayerIndex = (this.currentPlayerIndex + 1) % 2 
    },
    
    checkGameOver() {
      const opp = this.opponent
      return opp.board.flat().every(cell => cell !== 1)
    },
    
    // Инициализация кораблей для нового игрока
    initializeShips() {
      return [
        { size: 4, count: 1, placed: 0 }, 
        { size: 3, count: 2, placed: 0 }, 
        { size: 2, count: 3, placed: 0 }, 
        { size: 1, count: 4, placed: 0 }   
      ]
    },
    
    initializeBoard() {
      return Array(10).fill().map(() => Array(10).fill(0))
    }
  }
}
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
</style>