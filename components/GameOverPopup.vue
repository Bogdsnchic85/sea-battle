<template>
  <!-- Подложка для попапа -->
  <div class="game-over-overlay">
    
    <div class="game-over-popup">
      <h2>Игра окончена!</h2>
      <p>Победитель: {{ winner.name }}</p>
      <button @click="$emit('restart')" class="restart-btn">
        Новая игра
      </button>
    </div>
  </div>
</template>

<script setup>
// Победитель
defineProps({
  winner: {
    type: Object,
    required: true
  }
});

defineEmits(['restart']);
</script>

<style scoped>
.game-over-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.7); 
  display: flex;
  justify-content: center; 
  align-items: center; 
  z-index: 1000; 
}

/* Попап */
.game-over-popup {
  background-color: white;
  padding: 30px;
  border-radius: 10px;
  text-align: center;
  max-width: 400px;
  width: 100%;
}

/* Кнопка */
.restart-btn {
  margin-top: 20px;
  padding: 12px 24px;
  background-color: #4CAF50; 
  color: white;
  border: none;
  border-radius: 5px;
  font-size: 16px;
  cursor: pointer;
  transition: background-color 0.3s; 
}
.restart-btn:hover {
  background-color: #45a049; 
}
</style>