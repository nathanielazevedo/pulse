<template>
  <div class="dialog">
    <div class="dialog-content">
      <div style="display: flex; justify-content: space-between; align-items: center; padding-bottom: 8px; padding-right: 10px;">

        <h4>Select a Sample</h4>
        <button class="close-btn" @click="$emit('close')">✖</button>
      </div>
      <div class="sample-grid">
        <div v-for="sample in samples" :key="sample.name" class="sample-pad">
          <button class="pad-button" @click="$emit('play-sample', sample)">
            ▶ {{ sample.name }}
          </button>
          <button class="add-button" @click="$emit('add-sample', sample)">
            ➕ Add
          </button>
        </div>
      </div>
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
  background: rgba(0, 0, 0, 0.277);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  backdrop-filter: blur(4px);
}

.dialog-content {
  background: #1e1e1e;
  color: white;
  padding: 1rem;
  padding-right: 0;
  border-radius: 5px;
  width: 90%;
  max-width: 600px;
  max-height: 85vh;
  overflow-y: hidden;
  box-shadow: 0 0 20px rgba(255, 255, 255, 0.05);
    border: rgb(87, 87, 87) 1px solid;
}

h4 {
  /* text-align: left; */
  font-weight: 100;
  /* margin-bottom: 1rem; */
  color: grey;
  /* font-size: 1.5rem; */
}

.sample-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  gap: 1rem;
    max-height: 70vh;
      overflow-y: auto;
          padding-right: 12px;
}

.sample-pad {
  display: flex;
  flex-direction: column;
  align-items: center;
  background: #2a2a2a;
  padding: 1rem;
  border-radius: 5px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.4);
}

.pad-button,
.add-button {
  width: 100%;
  padding: 0.5rem;
  margin: 0.25rem 0;
  border: none;
  /* border-radius: 0.5rem; */
  /* background: linear-gradient(to bottom, #444, #333); */
  background-color: inherit;
  color: #fff;
  cursor: pointer;
  font-size: 0.9rem;
  transition: background 0.2s, box-shadow 0.2s;
}

/* .pad-button:hover,
.add-button:hover {
  background: linear-gradient(to bottom, #555, #444);
  box-shadow: 0 0 5px #fff3;
} */

.close-btn {

  border: none;
  background-color: inherit;


  font-size: 2rem;
  color: grey;
  cursor: pointer;

}

.sample-grid::-webkit-scrollbar {
  width: 10px;
}

.sample-grid::-webkit-scrollbar-track {
  background: #1e1e1e;
  border-radius: 10px;
}

.sample-grid::-webkit-scrollbar-thumb {
  background-color: #555;
  border-radius: 10px;
  border: 2px solid #1e1e1e;
}

.sample-grid::-webkit-scrollbar-thumb:hover {
  background-color: #777;
}

</style>
