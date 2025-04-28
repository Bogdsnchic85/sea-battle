<template>
  <div class="ship-tracker">
    <h3>🔫 Корабли противника</h3>
    <div class="ships-list">
      <div v-for="ship in enemyShips" :key="ship.size" class="ship-item">
        <div class="ship-info">
          <span class="ship-size">{{ ship.size }}-палубный:</span>
          <span class="ship-count">{{ ship.count - ship.sunk }} из {{ ship.count }}</span>
        </div>
        <div class="ship-progress">
          <div 
            v-for="i in ship.count" 
            :key="i"
            class="progress-cell"
            :class="{ 'sunk': i <= ship.sunk }"
          ></div>
        </div>
      </div>
    </div>
    
    <h3>Потопленные корабли:</h3>
    <div v-if="sunkShips.length === 0" class="no-ships">
      ~ Пока нет потопленных кораблей ~
    </div>
    <div v-else v-for="(ship, index) in sunkShips" :key="index" class="sunk-ship">
      <span class="sunk-icon">💥</span>
      <span class="sunk-size">{{ ship.length }}-палубный</span>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue';

const props = defineProps({
  player: Object,
  opponent: Object
});

const enemyShips = computed(() => {
  const sizes = [4, 3, 2, 1];
  return sizes.map(size => {
    const total = props.opponent.ships.find(s => s.size === size)?.count || 0;
    const sunk = props.player.sunkShips.filter(s => s.length === size).length;
    return { size, count: total, sunk };
  });
});

const sunkShips = computed(() => props.player.sunkShips || []);
</script>

<style scoped>
.ship-tracker {
  background: white;
  padding: 15px;
  border-radius: 5px;
  border: 1px solid #d3d3d3;
  min-width: 200px;
}

.ships-list {
  margin-bottom: 20px;
}

.ship-item {
  margin-bottom: 10px;
}

.ship-info {
  display: flex;
  justify-content: space-between;
  margin-bottom: 5px;
  font-size: 0.9em;
}

.ship-progress {
  display: flex;
  gap: 3px;
}

.progress-cell {
  width: 15px;
  height: 15px;
  border: 1px solid #ddd;
  border-radius: 2px;
}

.progress-cell.sunk {
  background-color: #f44336;
  border-color: #d32f2f;
}

.sunk-ship {
  display: flex;
  align-items: center;
  gap: 8px;
  margin: 5px 0;
  padding: 5px;
  background: #ffebee;
  border-radius: 3px;
}

.sunk-icon {
  font-size: 1.2em;
}

.sunk-size {
  color: #e74c3c;
  font-weight: 500;
}

.no-ships {
  color: #999;
  font-style: italic;
  text-align: center;
  padding: 10px;
}
</style>