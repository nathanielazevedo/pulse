<template>
  <div class="dialog">
    <div class="dialog-content">
      <h2>Select a Sample</h2>
      <div class="sample-grid">
        <div v-for="sample in samples" :key="sample.name" class="sample-pad">
          <button @click="$emit('play-sample', sample)">
            ▶ {{ sample.name }}
          </button>
          <button @click="$emit('add-sample', sample)">
            ➕ Add
          </button>
        </div>
      </div>
      <button class="close-btn" @click="$emit('close')">Close</button>
    </div>
  </div>
</template>

<script setup>
defineProps({
  samples: Array
})
defineEmits(['close', 'add-sample', 'play-sample'])
</script>

<style scoped>
.dialog {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.dialog-content {
  background: #2c2c2c;
  color: white;
  padding: 2rem;
  border-radius: 1rem;
  width: 80%;
  max-height: 90vh;
  overflow-y: auto;
  /* max-width: 600px; */
}

.sample-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
  margin-top: 1rem;
}

.sample-pad {
  display: flex;
  flex-direction: column;
  align-items: center;
}

button {
  background: #444;
  color: white;
  border: none;
  padding: 0.5rem;
  margin: 0.25rem;
  border-radius: 0.5rem;
  cursor: pointer;
}

button:hover {
  background: #666;
}

.close-btn {
  margin-top: 1rem;
  background: crimson;
}
</style>
