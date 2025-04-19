<template>
  <div class="app">
    <TopNav :isPlaying="isPlaying" @toggle-transport="toggleTransport" :bpm="bpm" />
    <SideNav :bpm="bpm" :isRecording="isRecording" :vocalUrl="vocalUrl" @start-recording="startRecording"
      @stop-recording="stopRecording" @open-sample-dialog="showSampleDialog = true" />
    <main class="main">
      <StepGrid :steps="steps" :currentStep="currentStep" @toggle-step="toggleStep" />
    </main>
    <SampleDialog v-if="showSampleDialog" :samples="allSamples" @close="showSampleDialog = false"
      @add-sample="addSampleToGrid" @play-sample="playSample" />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import * as Tone from 'tone'
import TopNav from './TopNav.vue'
import SideNav from './SideNav.vue'
import StepGrid from './StepGrid.vue'
import SampleDialog from './SampleDialog.vue'

// Load samples from folder
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
let players = []
// const players = allSamples.value.slice(0, rows).map(sample => new Tone.Player(sample.path).toDestination())

async function initPlayers() {
  console.log('initing players')
  const slice = allSamples.value.slice(0, rows)

  players.value = allSamples.value.map(sample => new Tone.Player(sample.path).toDestination())

}
const stepIndex = ref(0)

function scheduleTransport() {
  console.log('scheduling transport')
  Tone.Transport.scheduleRepeat((time) => {
    currentStep.value = stepIndex.value

    steps.value.forEach((row, rowIndex) => {
      const player = players.value[rowIndex]
      if (row[stepIndex.value] && player && player.loaded) {
        try {
          player.stop()
          player.start(time)
        } catch (err) {
          console.error(`Player at row ${rowIndex} failed to start at step ${stepIndex.value}:`, err)
        }
      }
    })

    if (vocalPlayer && vocalReady && stepIndex.value === 0) {
      vocalPlayer.start(time)
    }

    stepIndex.value = (stepIndex.value + 1) % cols
  }, '8n')

}

function toggleStep(row, col) {
  steps.value[row][col] = !steps.value[row][col]
}

const isPlaying = ref(false)
let initialized = false

async function toggleTransport() {
  console.log('toggling transport')
  await initPlayers()
  await Tone.start()

  if (!initialized) {
    scheduleTransport()
    initialized = true
  }

  Tone.Transport.bpm.value = bpm.value

  if (isPlaying.value) {
    Tone.Transport.stop()
    stepIndex.value = 0
    currentStep.value = 0
    if (vocalPlayer) vocalPlayer.stop()
  } else {
    Tone.Transport.start('+0.1')
  }

  isPlaying.value = !isPlaying.value
}







// Recording logic
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

// Sample dialog
const showSampleDialog = ref(false)

function addSampleToGrid(sample) {
  // 1. Add a new empty row to the grid
  steps.value.push(Array(cols).fill(false))

  // 2. Create a new player for the selected sample
  const newPlayer = new Tone.Player(sample.path).toDestination()

  // 3. Add it to the players list
  players.value.push(newPlayer)
}


function playSample(sample) {
  const tempPlayer = new Tone.Player({
    url: sample.path,
    onload: () => tempPlayer.start()
  }).toDestination()
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

.main {
  grid-area: main;
  padding: 0;
  overflow: hidden;
}

</style>
