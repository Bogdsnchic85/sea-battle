<!-- Смена хода -->

<template>
  <div class="fullscreen-overlay">
    <div class="turn-popup">
      <h2>Сейчас ходит:</h2>
      <div class="player-turn" :style="{ color: player.color }">
        {{ player.name }}
      </div>
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
.fullscreen-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-color: rgba(255, 255, 255, 0.95);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.turn-popup {
  background-color: white;
  padding: 2rem;
  border-radius: 10px;
  text-align: center;
  max-width: 400px;
  width: 90%;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
  border: 3px solid #f0f0f0;
}

.player-turn {
  font-size: 2rem;
  font-weight: bold;
  margin: 1rem 0;
  text-shadow: 1px 1px 2px rgba(0,0,0,0.1);
}

.start-turn-btn {
  margin-top: 1.5rem;
  padding: 0.8rem 2rem;
  background: #9C27B0;
  color: white;
  border: none;
  border-radius: 50px;
  font-size: 1.1rem;
  cursor: pointer;
  transition: all 0.3s;
  font-family: 'Schoolbell', cursive;
}

.start-turn-btn:hover {
  background: #7B1FA2;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}
</style>