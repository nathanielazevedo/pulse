<template>
  <aside class="side-nav">
    <div class="columns-input">
      <label for="cols-input">Steps:</label>
      <input min="1" max="64" v-model.number="columnsLocal" @change="updateColumns" />


    </div>

    <label class="bpm-slider">
      <div style="display: flex; justify-content: space-between;">
        BPM:
        <span>{{ bpm }}</span>
      </div>
      <input type="range" min="60" max="180" :value="bpm" @input="$emit('update:bpm', Number($event.target.value))" />
    </label>

    <button @click="$emit('open-sample-dialog')">Beats</button>
    <!-- <button>Vocals</button> -->

    <!-- <div class="vocals">
      <h3>Vocal Recorder</h3>
      <button @click="$emit('start-recording')" :disabled="isRecording">⏺ Start</button>
      <button @click="$emit('stop-recording')" :disabled="!isRecording">⏹ Stop</button>
      <audio v-if="vocalUrl" :src="vocalUrl" controls />
    </div> -->
  </aside>
</template>

<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  columns: Number
})
const emit = defineEmits(['update:columns'])

const columnsLocal = ref(props.columns)

watch(() => props.columns, (newVal) => {
  columnsLocal.value = newVal
})

function updateColumns() {
  emit('update:columns', columnsLocal.value)
}
</script>



<style scoped>
.side-nav {
  grid-area: side-nav;
  background-color: #2a2a2a;
  padding: 1rem;
  display: flex;
  flex-direction: column;
  gap: 1rem;
  border-right: 1px solid #444;
}


.side-nav button {
  background-color: #333;
  color: #eee;
  border: none;
  padding: 0.75rem;
  border-radius: 6px;
  cursor: pointer;
  text-align: left;
}

.side-nav button:hover {
  background-color: #444;
}

.bpm-slider {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  font-weight: bold;
  color: #ccc;
}

.bpm-slider input[type="range"] {
  -webkit-appearance: none;
  width: 100%;
  height: 14px;
  border-radius: 7px;
  background: repeating-linear-gradient(90deg,
      #999,
      #aaa 1px,
      #999 2px);
  /* Brushed effect */
  box-shadow:
    inset 0 0 2px #444,
    0 1px 1px rgba(255, 255, 255, 0.3);
  position: relative;
  outline: none;
  transition: background 0.2s ease-in-out;
}

/* Thumb styling - Chrome & Safari */
.bpm-slider input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: radial-gradient(circle at 30% 30%, #eee, #888);
  border: 1px solid #666;
  box-shadow:
    inset -1px -1px 2px rgba(255, 255, 255, 0.2),
    inset 1px 1px 2px rgba(0, 0, 0, 0.5),
    0 0 2px rgba(0, 0, 0, 0.7);
  cursor: pointer;
  transition: transform 0.1s ease;
}

.bpm-slider input[type="range"]::-webkit-slider-thumb:active {
  transform: scale(1.1);
}

/* Firefox support */
.bpm-slider input[type="range"]::-moz-range-thumb {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: radial-gradient(circle at 30% 30%, #eee, #888);
  border: 1px solid #666;
  box-shadow:
    inset -1px -1px 2px rgba(255, 255, 255, 0.2),
    inset 1px 1px 2px rgba(0, 0, 0, 0.5),
    0 0 2px rgba(0, 0, 0, 0.7);
  cursor: pointer;
}

.columns-input {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  font-weight: bold;
  color: #ccc;
}

.columns-input label {
  font-size: 0.9rem;
}

.columns-input input{
  background-color: #333;
  color: #eee;
  border: 1px solid #555;
  padding: 0.4rem 0.6rem;
  border-radius: 6px;
  outline: none;
  width: 100%;
  box-shadow:
    inset 0 0 2px #444,
    0 1px 1px rgba(255, 255, 255, 0.2);
  transition: border 0.2s, box-shadow 0.2s;
}

.columns-input input[type="number"]:focus {
  border-color: #888;
  box-shadow:
    0 0 5px rgba(255, 255, 255, 0.3),
    inset 0 0 2px #666;
}
</style>