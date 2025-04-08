<template>
  <div class="turn-popup-overlay">
    <div class="turn-popup">
      <h2>Сейчас ходит: <span :style="{ color: player.color }">{{ player.name }}</span></h2>
      <p>Подготовьтесь к атаке!</p>
      <button @click="$emit('start-turn')" class="start-turn-btn">
        Начать ход
      </button>
    </div>
  </div>
</template>

<script setup>
defineProps({
  player: {
    type: Object,
    required: true
  }
});

defineEmits(['start-turn']);
</script>

<style scoped>
.turn-popup-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  backdrop-filter: blur(2px);
}

.turn-popup {
  background-color: white;
  padding: 30px;
  border-radius: 5px;
  text-align: center;
  max-width: 400px;
  width: 100%;
  box-shadow: 0 3px 10px rgba(0,0,0,0.2);
  border: 1px solid #e0e0e0;
  font-family: 'Courier New', monospace;
}

h2 {
  margin: 0 0 15px 0;
  font-weight: normal;
  color: #333;
}

.start-turn-btn {
  margin-top: 20px;
  padding: 10px 20px;
  background-color: white;
  color: #333;
  border: 1px solid #333;
  border-radius: 3px;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.2s;
  font-family: 'Courier New', monospace;
}

.start-turn-btn:hover {
  background-color: #333;
  color: white;
}
</style>