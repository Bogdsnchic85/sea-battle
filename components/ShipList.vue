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
        <span class="ship-size">{{ ship.size }}-палубный</span>
        <span class="ship-count">({{ ship.count - ship.placed }} из {{ ship.count }})</span>
      </div>
      <div 
        class="ship-preview"
        :style="{ 
          width: previewWidth(ship),
          backgroundColor: previewColor(ship)
        }"
      ></div>
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  ships: {
    type: Array,
    required: true
  }
});

const emit = defineEmits(['select-ship']);

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

const previewColor = (ship) => {
  return ship.placed < ship.count ? '#555' : '#aaa';
};
</script>

<style scoped>
.ship-list {
  display: flex;
  flex-direction: column;
  gap: 15px;
  padding: 15px;
  background: #f8f8f8;
  border-radius: 8px;
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
  font-size: 0.9em;
}

.ship-size {
  font-weight: bold;
}

.ship-count {
  color: #666;
}

.ship-preview {
  height: 15px;
  border-radius: 3px;
  transition: all 0.2s;
}
</style>