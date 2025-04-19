<template>
  <div class="grid">
    <div v-for="(row, rowIndex) in steps" :key="rowIndex" class="row">
      <div class="row-label">{{ row.name }}</div>
      <button v-for="(step, colIndex) in row.steps" :key="colIndex" :class="{
        active: step,
        playing: colIndex === currentStep
      }" @click="$emit('toggle-step', rowIndex, colIndex)" />
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
  gap: 10px; height: 100%;
  width: 100%;
  padding: 20px;
  padding-left: 10px;
}

.row {
  display: flex;
  flex-grow: 1;
  gap: 10px;
}

.row-label{
  color: grey;
  font-size: 12px;
  max-width: 100px;
  min-width: 100px;
  padding-top: 2px;
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
    background-color: #029779;
  /* background: radial-gradient(circle at center,
      rgba(187, 186, 186, 0.8) 0%,
      rgba(129, 128, 128, 0.5) 10%,
      rgba(131, 129, 129, 0.2) 30%,
      rgba(131, 129, 129, 0.1) 50%,
      rgba(255, 255, 255, 0.0) 100%);
  background-color: #333; */
}

button.playing {
  outline: 2px solid #06528b;
}
</style>
