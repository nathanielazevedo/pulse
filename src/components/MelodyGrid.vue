<template>
  <div class="grid">
    <div v-for="(row, rowIndex) in melodySteps" :key="rowIndex" class="row">
      <div class="row-label">
        {{ row.note }}
      </div>

      <button v-for="(step, colIndex) in row.steps" :key="colIndex" :class="{
        active: step,
        playing: colIndex === currentStep
      }" @click="$emit('toggle-note', rowIndex, colIndex)" />
    </div>
  </div>
</template>

<script setup>
defineProps({
  melodySteps: Array,
  currentStep: Number
})
defineEmits(['toggle-note'])
</script>

<style scoped>
.grid {
  display: flex;
  flex-direction: column;
  justify-content: stretch;
  align-items: stretch;
  gap: 10px;
  height: 97%;
  width: 100%;
  padding: 20px 20px;
  padding-left: 10px;
}

.row {
  display: flex;
  flex-grow: 1;
  gap: 10px;
}

.row-label {
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
}

button.playing {
  outline: 2px solid #06528b;
}
</style>
