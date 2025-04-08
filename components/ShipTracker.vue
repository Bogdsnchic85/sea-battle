<!-- Список потопленных кораблей -->

<template>
  <div class="ship-tracker">
    <h3>🔫 Потопленные корабли</h3>
    <div class="ships-list">
      <div v-if="sunkShips.length === 0" class="no-ships">
        ~ Пока пусто ~
      </div>
      <div v-else v-for="(ship, index) in sunkShips" :key="index" class="ship-item">
        <div class="ship-header">
          <span class="ship-icon" :style="{ color: player.color }">■</span>
          <span class="ship-size">{{ ship.length }}-палубный</span>
        </div>
        <div class="coordinates">
          <span v-for="(coord, i) in ship" :key="i" class="coord">
            ({{ coord[0]+1 }},{{ coord[1]+1 }})
          </span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue';

const props = defineProps({
  player: Object,
  opponent: Object
});

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
}

.ships-list {
  max-height: 300px;
  overflow-y: auto;
  padding-right: 10px;
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
}

.ship-icon {
  font-size: 1.4em;
}

.ship-size {
  color: #444;
  font-weight: bold;
}

.coordinates {
  display: flex;
  flex-wrap: wrap;
  gap: 5px;
  font-size: 0.9em;
  color: #666;
}

.coord {
  background: #f5f5f5;
  padding: 2px 5px;
  border-radius: 3px;
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
}

::-webkit-scrollbar-thumb {
  background: #888;
  border-radius: 3px;
}
</style>