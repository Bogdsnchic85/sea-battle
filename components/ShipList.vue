<template>
  <div class="ship-list">
    <h3>Корабли для размещения:</h3>
    <div 
      v-for="(ship, index) in ships" 
      :key="index"
      class="ship-item"
      :class="{ 
        'selected': isSelected(ship),
        'disabled': ship.placed >= ship.count
      }"
      @click="selectShip(ship)"
    >
      <div class="ship-info">
        <span class="ship-size">{{ ship.size }} палубный</span>
        <span class="ship-count">({{ ship.count - ship.placed }} из {{ ship.count }})</span>
      </div>
      <div 
        class="ship-preview"
        :style="{ width: previewWidth(ship) }"
      ></div>
    </div>
  </div>
</template>

<script>
import { computed } from 'vue';

export default {
  props: {
    ships: {
      type: Array,
      required: true
    }
  },
  emits: ['select-ship'],
  setup(props, { emit }) {
    const isSelected = (ship) => {
      return ship.placed < ship.count;
    };

    const selectShip = (ship) => {
      if (ship.placed < ship.count) {
        emit('select-ship', { ...ship });
      }
    };

    const previewWidth = (ship) => {
      return `${ship.size * 20}px`;
    };

    return {
      isSelected,
      selectShip,
      previewWidth
    };
  }
}
</script>

<style scoped>
.ship-list {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.ship-item {
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  cursor: pointer;
  transition: all 0.2s;
}

.ship-item:hover {
  background-color: #f5f5f5;
}

.ship-item.selected {
  border-color: #4CAF50;
  background-color: #e8f5e9;
}

.ship-item.disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.ship-info {
  display: flex;
  justify-content: space-between;
  margin-bottom: 8px;
}

.ship-preview {
  height: 15px;
  background-color: #555;
  border-radius: 3px;
}
</style>