<!-- Конец игры -->

<template>
  <div class="fullscreen-overlay">
    <div class="game-over-popup">
      <h2>Игра окончена!</h2>
      <div class="winner-info">
        <div class="winner-label">Победитель:</div>
        <div class="winner-name" :style="{ color: winner.color }">
          {{ winner.name }}
        </div>
      </div>
      <div class="celebration">🏆</div>
      <button @click="$emit('restart')" class="restart-btn">
        Новая игра
      </button>
    </div>
  </div>
</template>

<script setup>
defineProps({
  winner: {
    type: Object,
    required: true
  }
});

defineEmits(['restart']);
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

.game-over-popup {
  background-color: white;
  padding: 2rem;
  border-radius: 10px;
  text-align: center;
  max-width: 500px;
  width: 90%;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
  border: 3px solid #FFD700;
}

.winner-info {
  margin: 1rem 0;
}

.winner-label {
  font-size: 1.2rem;
  color: #666;
}

.winner-name {
  font-size: 2rem;
  font-weight: bold;
  margin-top: 0.5rem;
}

.celebration {
  font-size: 4rem;
  margin: 1rem 0;
  animation: bounce 1s infinite alternate;
}

@keyframes bounce {
  from { transform: translateY(0); }
  to { transform: translateY(-20px); }
}

.restart-btn {
  margin-top: 1.5rem;
  padding: 0.8rem 2rem;
  background: #FF5722;
  color: white;
  border: none;
  border-radius: 50px;
  font-size: 1.1rem;
  cursor: pointer;
  transition: all 0.3s;
  font-family: 'Schoolbell', cursive;
}

.restart-btn:hover {
  background: #E64A19;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}
</style>