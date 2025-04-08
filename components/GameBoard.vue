<template>
  <div class="game-board">
    <div 
      v-for="(row, rowIndex) in board" 
      :key="rowIndex" 
      class="board-row"
    >
      <div
        v-for="(cell, colIndex) in row"
        :key="colIndex"
        class="board-cell"
        :class="{
          'has-ship': cell === 1,
          'hover-ship': isHovered(rowIndex, colIndex)
        }"
        @click="handleCellClick(rowIndex, colIndex)"
        @mouseover="hoverCell(rowIndex, colIndex)"
        @mouseleave="resetHover()"
      ></div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

const props = defineProps({
  board: Array,
  selectedShip: Object,
  shipOrientation: String
});

const emit = defineEmits(['cell-click']);

const hoverPosition = ref({ row: -1, col: -1 });

const isHovered = (row, col) => {
  if (!props.selectedShip || hoverPosition.value.row === -1) return false;
  
  const size = props.selectedShip.size;
  const orientation = props.shipOrientation;
  
  if (orientation === 'horizontal') {
    return row === hoverPosition.value.row && 
           col >= hoverPosition.value.col && 
           col < hoverPosition.value.col + size;
  } else {
    return col === hoverPosition.value.col && 
           row >= hoverPosition.value.row && 
           row < hoverPosition.value.row + size;
  }
};

const hoverCell = (row, col) => {
  if (props.selectedShip) {
    hoverPosition.value = { row, col };
  }
};

const resetHover = () => {
  hoverPosition.value = { row: -1, col: -1 };
};

const handleCellClick = (row, col) => {
  emit('cell-click', row, col);
};
</script>

<style scoped>
.game-board {
  border: 2px solid #333;
  display: inline-block;
}

.board-row {
  display: flex;
}

.board-cell {
  width: 30px;
  height: 30px;
  border: 1px solid #ccc;
  background-color: #f0f8ff;
  cursor: pointer;
}

.board-cell.has-ship {
  background-color: #555;
}

.board-cell.hover-ship {
  background-color: rgba(85, 85, 85, 0.5);
}
</style>