<!-- Результат атаки -->

<template>
  <div class="fullscreen-overlay">
    <div class="hit-result-popup" :class="resultClass">
      <div class="result-icon">
        {{ resultIcon }}
      </div>
      <h3>{{ result.message }}</h3>
      <button @click="$emit('continue')" class="continue-btn">
        Продолжить
      </button>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue';

const props = defineProps({
  result: {
    type: Object,
    required: true
  }
});

const emit = defineEmits(['continue']);

const resultClass = computed(() => ({
  'hit': props.result.hit,
  'miss': !props.result.hit,
  'sunk': props.result.shipDestroyed
}));

const resultIcon = computed(() => {
  if (props.result.shipDestroyed) return '💥';
  return props.result.hit ? '🔥' : '💦';
});
</script>

<style scoped>
.fullscreen-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.hit-result-popup {
  background-color: white;
  padding: 2rem;
  border-radius: 10px;
  text-align: center;
  max-width: 400px;
  width: 90%;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
  transition: all 0.3s;
}

.hit-result-popup.hit {
  border: 3px solid #4CAF50;
}

.hit-result-popup.miss {
  border: 3px solid #F44336;
}

.hit-result-popup.sunk {
  border: 3px solid #FF9800;
}

.result-icon {
  font-size: 3rem;
  margin-bottom: 1rem;
  line-height: 1;
}

.continue-btn {
  margin-top: 1.5rem;
  padding: 0.8rem 2rem;
  background: #FFC107;
  color: #333;
  border: none;
  border-radius: 50px;
  font-size: 1.1rem;
  cursor: pointer;
  transition: all 0.3s;
  font-family: 'Schoolbell', cursive;
  font-weight: bold;
}

.continue-btn:hover {
  background: #FFA000;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}
</style>