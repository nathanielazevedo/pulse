<template>
  <div class="grid">
    <div v-for="(row, rowIndex) in steps" :key="rowIndex" class="row">
      <button v-for="(step, colIndex) in row" :key="colIndex"
        :class="{ active: step, playing: colIndex === currentStep }"
        @click="$emit('toggle-step', rowIndex, colIndex)" />
    </div>
  </div>
</template>

<script setup>
defineProps({
  steps: Array,
  currentStep: Number
})
</script>

<style scoped>
.grid {
  display: flex;
  flex-direction: column;
  justify-content: stretch;
  align-items: stretch;
  gap: 10px;
  height: 100%;
  width: 100%;
  padding: 20px;
}

.row {
  display: flex;
  flex-grow: 1;
  gap: 10px;
}

.row button {
  flex-grow: 1;
  flex-basis: 0;
  border: none;
  background: #333;
  cursor: pointer;
  border-radius: 6px;
  transition: background 0.2s;
  width: 100%;
}

button.active {
  background-color: #00b894;
}

button.playing {
  outline: 2px solid #06528b;
}
</style>
