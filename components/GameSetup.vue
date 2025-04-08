<!-- Настройки игры -->

<template>
  <div class="game-setup">
    <h1>⚙️ Настройка игры</h1>
    
    <div class="players-setup">
      <!-- Настройка игрока 1 -->
      <div 
        class="player-input" 
        :style="{ borderColor: players[0].color }"
      >
        <label>👤 Игрок 1:</label>
        <input 
          v-model="players[0].name" 
          placeholder="Введите имя" 
          class="pencil-input"
        />
        <div class="color-picker">
          <span>🎨 Цвет:</span>
          <input 
            type="color" 
            v-model="players[0].color" 
            class="color-input"
          />
        </div>
      </div>
      
      <!-- Настройка игрока 2 -->
      <div 
        class="player-input" 
        :style="{ borderColor: players[1].color }"
      >
        <label>👤 Игрок 2:</label>
        <input 
          v-model="players[1].name" 
          placeholder="Введите имя" 
          class="pencil-input"
        />
        <div class="color-picker">
          <span>🎨 Цвет:</span>
          <input 
            type="color" 
            v-model="players[1].color" 
            class="color-input"
          />
        </div>
      </div>
    </div>
    
    <button @click="startGame" class="start-button">
      🚀 Начать игру!
    </button>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const emit = defineEmits(['game-start']);

const players = ref([
  { name: 'Игрок 1', color: '#3366ff' }, 
  { name: 'Игрок 2', color: '#ff3333' }  
]);

const startGame = () => {
  if (players.value[0].name.trim() === '' || players.value[1].name.trim() === '') {
    alert('Пожалуйста, введите имена обоих игроков');
    return;
  }
  emit('game-start', players.value);
};
</script>

<style scoped>
.game-setup {
  background: white;
  padding: 30px;
  border-radius: 10px;
  box-shadow: 0 3px 10px rgba(0,0,0,0.1);
  font-family: 'Schoolbell', cursive;
}

.players-setup {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 30px;
  margin: 30px 0;
}

.player-input {
  padding: 20px;
  border: 2px solid;
  border-radius: 8px;
  background: #f8f8f8;
}

.pencil-input {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-family: 'Schoolbell', cursive;
  font-size: 16px;
}

.color-picker {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-top: 15px;
}

.color-input {
  width: 40px;
  height: 40px;
  border: 2px solid #333;
  border-radius: 50%;
  cursor: pointer;
}

.start-button {
  padding: 15px 40px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 25px;
  font-size: 1.2em;
  cursor: pointer;
  transition: all 0.2s;
  font-family: 'Schoolbell', cursive;
}

.start-button:hover {
  transform: scale(1.05);
  box-shadow: 0 3px 10px rgba(0,0,0,0.2);
}
</style>