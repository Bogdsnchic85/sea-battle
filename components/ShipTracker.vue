<!-- Трекер кораблей - список потоплений -->

<template>
  <div class="ship-tracker">
    <h3>🔫 Потопленные корабли</h3>
    <div class="ships-list">
      <div v-if="sunkShips.length === 0" class="no-ships">
        ~ Пока пусто ~
      </div>
      <template v-else>
        <div 
          v-for="(ship, index) in sunkShips" 
          :key="index" 
          class="ship-item"
        >
          <div class="ship-header">
            <span class="ship-icon" :style="{ color: player.color }">■</span>
            <span class="ship-size">{{ ship.length }}-палубный</span>
            <span class="ship-coords-count">{{ ship.length }} клетки</span>
          </div>
          <div class="coordinates">
            <span 
              v-for="(coord, i) in ship" 
              :key="i" 
              class="coord"
              :title="`(${coord[0]+1}, ${coord[1]+1})`"
            >
              {{ String.fromCharCode(65 + coord[1]) }}{{ coord[0]+1 }}
            </span>
          </div>
        </div>
      </template>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue';

const props = defineProps({
  player: {
    type: Object,
    required: true
  },
  opponent: {
    type: Object,
    default: null
  }
});
// отоброжение потоплений
const sunkShips = computed(() => props.player.sunkShips || []);
</script>

<style scoped>
.ship-tracker {
  background: white;
  padding: 15px;
  border-radius: 5px;
  border: 1px solid #d3d3d3;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
  font-family: 'Schoolbell', cursive;
  min-width: 250px;
  max-height: 500px;
  display: flex;
  flex-direction: column;
}

h3 {
  margin-top: 0;
  text-align: center;
}

.ships-list {
  flex: 1;
  overflow-y: auto;
  padding-right: 5px;
}

.ship-item {
  margin: 10px 0;
  padding: 10px;
  border-bottom: 1px dashed #ddd;
}

.ship-header {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 5px;
  flex-wrap: wrap;
}

.ship-icon {
  font-size: 1.2em;
  line-height: 1;
}

.ship-size {
  color: #444;
  font-weight: bold;
  font-size: 0.9em;
}

.ship-coords-count {
  font-size: 0.8em;
  color: #666;
  margin-left: auto;
}

.coordinates {
  display: flex;
  flex-wrap: wrap;
  gap: 5px;
  font-size: 0.8em;
}

.coord {
  background: #f5f5f5;
  padding: 2px 5px;
  border-radius: 3px;
  cursor: default;
}

.no-ships {
  color: #999;
  text-align: center;
  padding: 10px;
  font-style: italic;
}

/* Стили для скроллбара */
::-webkit-scrollbar {
  width: 6px;
}

::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 3px;
}

::-webkit-scrollbar-thumb {
  background: #888;
  border-radius: 3px;
}

::-webkit-scrollbar-thumb:hover {
  background: #555;
}
</style>