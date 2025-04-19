<template>
  <div class="app">
    <TopNav :isPlaying="isPlaying" @toggle-transport="toggleTransport" />
    <SideNav :bpm="bpm" :isRecording="isRecording" :vocalUrl="vocalUrl" @start-recording="startRecording"
      @stop-recording="stopRecording" @open-sample-dialog="showSampleDialog = true" @update:bpm="bpm = $event" />
    <main class="main">
      <StepGrid :steps="steps" :currentStep="currentStep" @toggle-step="toggleStep" @remove-row="removeRow" />

    </main>
    <SampleDialog v-if="showSampleDialog" :grouped-samples="groupedSamples" @close="showSampleDialog = false"
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
const rawSampleModules = import.meta.glob('../assets/samples/**/*.wav', { eager: true })
console.log(rawSampleModules)
const allSamples = ref(
  Object.entries(rawSampleModules).map(([path, module]) => {
    const segments = path.split('/')
    const category = segments[segments.length - 2] // folder name
    const fileName = segments[segments.length - 1]
    const name = fileName.replace('.wav', '')
    return { name, path: module.default, category }
  })
)

import { computed } from 'vue'

const groupedSamples = computed(() => {
  const groups = {}
  for (const sample of allSamples.value) {
    if (!groups[sample.category]) {
      groups[sample.category] = []
    }
    groups[sample.category].push(sample)
  }
  return groups
})




const rows = 4
const cols = 8
const steps = ref(
  allSamples.value.slice(0, rows).map(sample => ({
    name: sample.name,
    steps: Array(cols).fill(false),
  }))
)

const bpm = ref(120)
const currentStep = ref(0)
let players = []

console.log(bpm)
async function initPlayers() {
  console.log('initing players')

  players.value = steps.value.map(row => {
    const sample = allSamples.value.find(s => s.name === row.name)
    if (!sample) {
      console.warn(`Sample not found for: ${row.name}`)
      return null
    }
    return new Tone.Player(sample.path).toDestination()
  })
}

const stepIndex = ref(0)

function scheduleTransport() {
  Tone.Transport.scheduleRepeat((time) => {
    currentStep.value = stepIndex.value

    steps.value.forEach((rowObj, rowIndex) => {
      const player = players.value[rowIndex]
      if (rowObj.steps[stepIndex.value] && player && player.loaded) {
        try {
          player.stop()
          player.start(time)
        } catch (err) {
          console.error(`Player ${rowObj.name} at row ${rowIndex} failed:`, err)
        }
      }
    })


    if (vocalPlayer && vocalReady && stepIndex.value === 0) {
      vocalPlayer.start(time)
    }

    stepIndex.value = (stepIndex.value + 1) % cols
  }, '8n')

}

function toggleStep(rowIndex, colIndex) {
  steps.value[rowIndex].steps[colIndex] = !steps.value[rowIndex].steps[colIndex]
}


const isPlaying = ref(false)
let initialized = false

async function toggleTransport() {
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
  steps.value.push({
    name: sample.name,
    steps: Array(cols).fill(false),
  })

  const newPlayer = new Tone.Player(sample.path).toDestination()
  players.value.push(newPlayer)
}



function playSample(sample) {
  const tempPlayer = new Tone.Player({
    url: sample.path,
    onload: () => tempPlayer.start()
  }).toDestination()
}

function removeRow(rowIndex) {
  // Stop the player if it's currently running
  const player = players[rowIndex]
  if (player && player.state === 'started') {
    player.stop()
  }

  // Remove player and step row
  players.splice(rowIndex, 1)
  steps.value.splice(rowIndex, 1)
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
