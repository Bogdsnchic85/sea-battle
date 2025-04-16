<!-- Попап перед расстановкой -->

<template>
  <div class="fullscreen-overlay">
    <div class="placement-start-popup">
      <h2>{{ player.name }}, расставьте ваши корабли!</h2>
      <div class="instructions">
        <p>Используйте мышку для размещения кораблей на поле</p>
        <ul class="tips">
          <li>1. Выберите корабль из списка</li>
          <li>2. Кликните на поле для размещения</li>
          <li>3. Нажмите ПРОБЕЛ для поворота</li>
        </ul>
      </div>
      <button @click="$emit('start-placement')" class="start-btn">
        Начать расстановку
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

defineEmits(['start-placement']);
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

.placement-start-popup {
  background-color: white;
  padding: 2rem;
  border-radius: 10px;
  text-align: center;
  max-width: 500px;
  width: 90%;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
  border: 3px solid #2196F3;
}

.instructions {
  margin: 1.5rem 0;
  text-align: left;
  background: #f5f5f5;
  padding: 1rem;
  border-radius: 5px;
}

.tips {
  margin-top: 0.5rem;
  padding-left: 1.5rem;
  color: #555;
}

.tips li {
  margin-bottom: 0.5rem;
}

.start-btn {
  margin-top: 1.5rem;
  padding: 0.8rem 2rem;
  background: #2196F3;
  color: white;
  border: none;
  border-radius: 50px;
  font-size: 1.1rem;
  cursor: pointer;
  transition: all 0.3s;
  font-family: 'Schoolbell', cursive;
}

.start-btn:hover {
  background: #1976D2;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}
</style>