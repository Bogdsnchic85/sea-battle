<!-- Настройка игры -->

<template>
  <div class="game-setup">
    <h1>⚙️ Настройка игры</h1>
    
    <div class="players-setup">
      <div 
        v-for="(player, index) in players" 
        :key="index" 
        class="player-input"
        :style="{ borderColor: player.color }"
      >
        <label>👤 Игрок {{ index + 1 }}:</label>
        <input
          v-model="player.name"
          placeholder="Введите имя"
          class="pencil-input"
        />
        <div class="color-picker">
          <span>🎨 Цвет:</span>
          <input
            type="color"
            v-model="player.color"
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
 //логика формы
const players = ref([
  { name: 'Игрок 1', color: '#3366ff' },
  { name: 'Игрок 2', color: '#ff3333' }
]);

//передача данных
const startGame = () => {
  if (players.value.some(p => p.name.trim() === '')) {
    alert('Пожалуйста, введите имена обоих игроков');
    return;
  }
  emit('game-start', players.value.map(p => ({ ...p })));
};
</script>

<style scoped>
.game-setup {
  background: white;
  padding: 30px;
  border-radius: 10px;
  box-shadow: 0 3px 10px rgba(0,0,0,0.1);
  font-family: 'Arial', sans-serif;
  max-width: 800px;
  margin: 0 auto;
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
  font-size: 16px;
  margin-top: 8px;
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
  display: block;
  margin: 0 auto;
}

.start-button:hover {
  background: #388E3C;
}

@media (max-width: 768px) {
  .players-setup {
    grid-template-columns: 1fr;
  }
}
</style>