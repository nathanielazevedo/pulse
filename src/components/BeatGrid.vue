<template>
  <div class="app">
    <header class="top-nav">
      <h2 style="font-weight: 100; color: grey;">Pulse</h2>
      <div class="controls">
        <button class="metal-button" @click="toggleTransport" title="Play / Pause">
          <span v-if="isPlaying">⏹</span>
          <span v-else>▶</span>
        </button>

      </div>
    </header>

    <aside class="side-nav">
      <label class="bpm-slider">BPM:
        <input type="range" min="60" max="180" v-model="bpm" />
        <span>{{ bpm }}</span>
      </label>

      <button @click="showSampleDialog = true">Beats</button>
      <button>Vocals</button>


      <div class="vocals">
        <h3>Vocal Recorder</h3>
        <button @click="startRecording" :disabled="isRecording">⏺ Start</button>
        <button @click="stopRecording" :disabled="!isRecording">⏹ Stop</button>
        <audio v-if="vocalUrl" :src="vocalUrl" controls />
      </div>
    </aside>

    <main class="main">
      <div class="grid">
        <div v-for="(row, rowIndex) in steps" :key="rowIndex" class="row">
          <button v-for="(step, colIndex) in row" :key="colIndex"
            :class="{ active: step, playing: colIndex === currentStep }" @click="toggleStep(rowIndex, colIndex)" />
        </div>
      </div>
    </main>

    <!-- Sample Browser Dialog -->
    <div v-if="showSampleDialog" class="dialog-overlay" @click.self="showSampleDialog = false">
      <div class="dialog">
        <h3>Select a Sample</h3>
        <div class="pad-grid">
          <button v-for="(sample, index) in allSamples" :key="index" @click="playSample(sample)">
            {{ sample.name }}
          </button>
        </div>
        <button class="metal-button" @click="addSampleToGrid">Add to Grid</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import * as Tone from 'tone'
const rawSampleModules = import.meta.glob('/src/assets/samples/*.wav', { eager: true })

const allSamples = ref(
  Object.entries(rawSampleModules).map(([path, module]) => {
    const name = path.split('/').pop().replace('.wav', '')
    return { name, path: module.default }
  })
)
const rows = 4
const cols = 8
const steps = ref(Array.from({ length: rows }, () => Array(cols).fill(false)))
const bpm = ref(120)
const currentStep = ref(0)
const players = allSamples.value.slice(0, rows).map(sample => new Tone.Player(sample.path).toDestination())
// const samplePaths = [
//   '/src/assets/samples/DD5_Kick_01.wav',
//   '/src/assets/samples/DD5_Snare_01.wav',
//   '/src/assets/samples/DD5_Shk_01.wav',
//   '/src/assets/samples/DD5_Perc_01.wav',
// ]

// const players = samplePaths.map(path => new Tone.Player(path).toDestination())

let stepIndex = 0
Tone.Transport.scheduleRepeat((time) => {
  currentStep.value = stepIndex
  steps.value.forEach((row, rowIndex) => {
    if (row[stepIndex]) {
      players[rowIndex].start(time)
    }
  })
  if (vocalPlayer && vocalReady && stepIndex === 0) {
    vocalPlayer.start(time)
  }
  stepIndex = (stepIndex + 1) % cols
}, '8n')

function toggleStep(row, col) {
  steps.value[row][col] = !steps.value[row][col]
}

const isPlaying = ref(false)

async function toggleTransport() {
  await Tone.start()
  Tone.Transport.bpm.value = bpm.value

  if (isPlaying.value) {
    Tone.Transport.stop()
    stepIndex = 0
    currentStep.value = 0
    if (vocalPlayer) vocalPlayer.stop()
  } else {
    Tone.Transport.start()
  }

  isPlaying.value = !isPlaying.value
}


// 🎤 Recording logic
const isRecording = ref(false)
const vocalChunks = []
let mediaRecorder = null
const vocalUrl = ref(null)
let vocalPlayer = null
let vocalReady = false

async function startRecording() {
  const stream = await navigator.mediaDevices.getUserMedia({ audio: true })
  mediaRecorder = new MediaRecorder(stream)
  vocalChunks.length = 0
  mediaRecorder.ondataavailable = e => vocalChunks.push(e.data)
  mediaRecorder.onstop = () => {
    const blob = new Blob(vocalChunks, { type: 'audio/webm' })
    vocalUrl.value = URL.createObjectURL(blob)
    vocalPlayer = new Tone.Player(vocalUrl.value).toDestination()
    vocalReady = true
  }
  mediaRecorder.start()
  isRecording.value = true
}

function stopRecording() {
  mediaRecorder?.stop()
  isRecording.value = false
}
</script>

<style scoped>
.app {
  display: grid;
  grid-template-columns: 200px 1fr;
  grid-template-rows: 60px 1fr;
  grid-template-areas:
    "top-nav top-nav"
    "side-nav main";
  height: 100vh;
  background-color: #1e1e1e;
  color: #fff;
  font-family: 'Segoe UI', sans-serif;
}

.top-nav {
  grid-area: top-nav;
  background-color: #2c2c2c;
  padding: 0 1rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-bottom: 1px solid #444;
}

.controls {
  display: flex;
  align-items: center;
  gap: 1rem;
}

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

.main {
  grid-area: main;
  padding: 0;
  overflow: hidden;
}

.grid {
  display: flex;
  flex-direction: column;
  justify-content: stretch;
  align-items: stretch;
  gap: 10px;
  height: 100%;
  width: 100%;
  padding: 20px
}

.row {
  display: flex;
  flex-grow: 1;
  gap: 10px;
}

.row button {
  flex-grow: 1;
  flex-basis: 0;
  /* aspect-ratio: 1 / 1; */
  border: none;
  background: #333;
  cursor: pointer;
  border-radius: 6px;
  transition: background 0.2s;
  width: 100%;
  /* ensures full width inside row */
}

/* button {
  width: 60px;
  height: 60px;
  border: none;
  background: #333;
  cursor: pointer;
  border-radius: 6px;
  transition: background 0.2s;
} */

button.active {
  background-color: #00b894;
}

button.playing {
  outline: 2px solid #0984e3;
}

.vocals {
  margin-top: 2rem;
}

audio {
  margin-top: 1rem;
}

.controls {
  display: flex;
  align-items: center;
}

.controls label {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-weight: bold;
  color: #ccc;
}

.controls input[type="range"] {
  -webkit-appearance: none;
  width: 120px;
  height: 6px;
  border-radius: 4px;
  background: #555;
  outline: none;
}

.controls input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 14px;
  height: 14px;
  border-radius: 50%;
  background: #00b894;
  cursor: pointer;
  box-shadow: 0 0 2px #000;
}



.controls button {
  color: #1e1e1e;
  padding: 0.6rem 0.8rem;
  border-radius: 6px;
  cursor: pointer;
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

.metal-button {
  width: 45px;
  height: 45px;
  border: none;
  border-radius: 50%;
  background: radial-gradient(circle at 30% 30%, #eee, #999);
  box-shadow:
    inset -2px -2px 3px rgba(255, 255, 255, 0.6),
    inset 2px 2px 5px rgba(0, 0, 0, 0.4),
    0 4px 6px rgba(0, 0, 0, 0.4);
  color: #222;
  font-size: 1.5rem;
  font-weight: bold;
  cursor: pointer;
  /* transition: transform 0.1s ease, box-shadow 0.1s ease; */
}

/* .metal-button:hover {
  transform: scale(1.05);
} */

.metal-button:active {
  transform: scale(0.98);
  box-shadow:
    inset 1px 1px 2px rgba(0, 0, 0, 0.6),
    inset -1px -1px 2px rgba(255, 255, 255, 0.2);
}

/* Optional: differentiate STOP button */
.metal-button.stop {
  background: radial-gradient(circle at 30% 30%, #f88, #c33);
  color: #fff;
  box-shadow:
    inset -2px -2px 2px rgba(255, 200, 200, 0.4),
    inset 2px 2px 3px rgba(0, 0, 0, 0.4),
    0 4px 6px rgba(0, 0, 0, 0.4);
}

.metal-button.stop:hover {
  transform: scale(1.05);
}

.metal-button.stop:active {
  transform: scale(0.98);
  box-shadow:
    inset 1px 1px 2px rgba(0, 0, 0, 0.6),
    inset -1px -1px 2px rgba(255, 200, 200, 0.2);
}

.dialog-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.dialog {
  background-color: #2c2c2c;
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 0 15px rgba(0, 0, 0, 0.5);
  color: white;
  max-width: 500px;
  width: 90%;
}

.pad-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
  gap: 1rem;
  margin: 1rem 0;
}

.pad-grid button {
  background: #444;
  color: #fff;
  padding: 1rem;
  border: none;
  border-radius: 8px;
  cursor: pointer;
}

.pad-grid button:hover {
  background: #666;
}
</style>
