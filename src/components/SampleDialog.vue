<template>
  <div class="dialog">
    <div class="dialog-content">
      <div class="dialog-header">
        <h4>Select a Sample</h4>
        <div class="category-select-wrapper">
          <label>
            <select v-model="selectedCategory" class="category-select">
              <option v-for="cat in categories" :key="cat" :value="cat">{{ cat }}</option>
            </select>
          </label>
        </div>
        <button class="close-btn" @click="$emit('close')">×</button>

      </div>


      <div class="grouped-samples">
        <div v-for="(samples, category) in filteredGroups" :key="category" class="sample-group">
          <h5 class="group-label">{{ category }}</h5>
          <div class="sample-grid">
            <div v-for="sample in samples" :key="sample.name" class="sample-pad">
              <button class="pad-button" @click="$emit('play-sample', sample)">
                <div> ▶ </div>{{ sample.name }}
              </button>
              <button class="add-button" @click="$emit('add-sample', sample)">
                Add to pad
              </button>
            </div>
          </div>
        </div>
      </div>

    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  groupedSamples: Object // { Bassdrums: [...], Snaredrums: [...], etc. }
})

defineEmits(['close', 'add-sample', 'play-sample'])

const selectedCategory = ref('All')

const categories = computed(() => {
  return ['All', ...Object.keys(props.groupedSamples)]
})

const filteredGroups = computed(() => {
  if (selectedCategory.value === 'All') {
    return props.groupedSamples
  }
  return {
    [selectedCategory.value]: props.groupedSamples[selectedCategory.value] || []
  }
})
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
  max-width: 90vw;
  max-height: 85vh;
  overflow-y: hidden;
  box-shadow: 0 0 20px rgba(255, 255, 255, 0.05);
  border: rgb(87, 87, 87) 1px solid;
}

.dialog-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-bottom: 8px;
  padding-right: 10px;
}

h4 {
  font-weight: 100;
  color: grey;
}

.grouped-samples {
  max-height: 70vh;
  overflow-y: auto;
  padding-right: 12px;
}

.group-label {
  font-weight: bold;
  margin: 1rem 0 0.5rem;
  text-transform: uppercase;
  font-size: 0.9rem;
  color: #ccc;
}

.sample-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  gap: 1rem;
}

.sample-pad {
  display: flex;
  flex-direction: column;
  align-items: center;
  background: #2a2a2a;
  padding: 2px;
  border-radius: 5px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.4);
}



.pad-button,
.add-button {
  width: 100%;
  padding: 0.5rem;
  margin: 0.25rem 0;
  border: none;
  background-color: inherit;
  color: #fff;
  cursor: pointer;
  font-size: 0.85rem;
  /* border-radius: 6px; */
  transition: background 0.2s ease, transform 0.1s ease;
  /* box-shadow: inset 0 0 4px #000; */
  overflow: hidden;
  font-size: 12px;
}

.pad-button:hover,
.add-button:hover {
  background-color: #444;
  transform: translateY(-1px);
}

.close-btn {
  border: none;
  background-color: inherit;
  font-size: 2rem;
  color: grey;
  cursor: pointer;
}

.grouped-samples::-webkit-scrollbar {
  width: 10px;
}

.grouped-samples::-webkit-scrollbar-track {
  background: #1e1e1e;
  border-radius: 10px;
}

.grouped-samples::-webkit-scrollbar-thumb {
  background-color: #555;
  border-radius: 10px;
  border: 2px solid #1e1e1e;
}

.grouped-samples::-webkit-scrollbar-thumb:hover {
  background-color: #777;
}

.category-select-wrapper {
  margin-left: auto;
  margin-right: 1rem;
}

.category-select {
  background-color: #2a2a2a;
  color: white;
  border: 1px solid #555;
  padding: 4px 8px;
  font-size: 0.9rem;
  border-radius: 4px;
}
</style>
