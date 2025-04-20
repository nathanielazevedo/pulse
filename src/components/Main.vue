<template>
  <div class="app">
    <TopNav :isPlaying="isPlaying" :activeTab="activeTab" @toggle-transport="toggleTransport"
      @update:activeTab="activeTab = $event" />

    <SideNav :bpm="bpm" :isRecording="isRecording" :vocalUrl="vocalUrl" @start-recording="startRecording"
      @stop-recording="stopRecording" @open-sample-dialog="showSampleDialog = true" @update:bpm="bpm = $event"
      v-model:columns="cols" />



    <main class="main">
      <template v-if="activeTab === 'drums'">
        <StepGrid :steps="steps" :currentStep="currentStep" @toggle-step="toggleStep" @remove-row="removeRow" />
      </template>

      <template v-else>
        <div class="melody-controls">
          <!-- <label for="synth-select">Melody Instrument:</label> -->
          <select v-model="selectedSynth">
            <option v-for="opt in synthOptions" :key="opt.value" :value="opt.value">
              {{ opt.label }}
            </option>
          </select>

        </div>

        <MelodyGrid :melodySteps="melodySteps" :currentStep="currentStep" @toggle-note="toggleMelodyStep" />
      </template>


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
import MelodyGrid from './MelodyGrid.vue'

const synthOptions = [
  { label: 'Classic Synth', value: 'Synth' },
  { label: 'Pluck Synth', value: 'PluckSynth' },
  { label: 'Duo Synth', value: 'DuoSynth' },
  { label: 'Mono Synth', value: 'MonoSynth' },
  { label: 'FM Synth', value: 'FMSynth' },
  { label: 'AM Synth', value: 'AMSynth' },
  { label: 'Membrane Synth', value: 'MembraneSynth' },
  { label: 'Piano (Sampler)', value: 'PianoSampler' },
  { label: 'Guitar (Sampler)', value: 'GuitarSampler' },
]

const selectedSynth = ref('PluckSynth')



const activeTab = ref('drums')

// Load samples from folder
const rawSampleModules = import.meta.glob('../assets/samples/**/*.wav', { eager: true })


let melodySynth = null

function createMelodySynth(type) {
  if (melodySynth) melodySynth.dispose()

  switch (type) {
    case 'PluckSynth':
      melodySynth = new Tone.PluckSynth().toDestination()
      break
    case 'DuoSynth':
      melodySynth = new Tone.DuoSynth().toDestination()
      break
    case 'MonoSynth':
      melodySynth = new Tone.MonoSynth().toDestination()
      break
    case 'FMSynth':
      melodySynth = new Tone.FMSynth().toDestination()
      break
    case 'AMSynth':
      melodySynth = new Tone.AMSynth().toDestination()
      break
    case 'MembraneSynth':
      melodySynth = new Tone.MembraneSynth().toDestination()
      break
    case 'PianoSampler':
      melodySynth = new Tone.Sampler({
        C4: '/samples/piano/C4.wav',
        D4: '/samples/piano/D4.wav',
      }).toDestination()

    case 'GuitarSampler':
      melodySynth = new Tone.Sampler({
        C4: '/samples/guitar/C4.wav',
      }).toDestination()
      break
    case 'Synth':
    default:
      melodySynth = new Tone.Synth().toDestination()
      break
  }
}


watch(selectedSynth, (newType) => {
  createMelodySynth(newType)
})

// Create it once on init
createMelodySynth(selectedSynth.value)


const allSamples = ref(
  Object.entries(rawSampleModules).map(([path, module]) => {
    const segments = path.split('/')
    const category = segments[segments.length - 2] // folder name
    const fileName = segments[segments.length - 1]
    const name = fileName.replace('.wav', '')
    return { name, path: module.default, category }
  })
)

const cols = ref(8)
// MELODY
const melodySteps = ref([
  { note: 'C4', steps: Array(cols.value).fill(false) },
  { note: 'D4', steps: Array(cols.value).fill(false) },
  { note: 'E4', steps: Array(cols.value).fill(false) },
  { note: 'F4', steps: Array(cols.value).fill(false) },
  { note: 'G4', steps: Array(cols.value).fill(false) },
  { note: 'A4', steps: Array(cols.value).fill(false) },
  { note: 'B4', steps: Array(cols.value).fill(false) },
])

function toggleMelodyStep(row, col) {
  melodySteps.value[row].steps[col] = !melodySteps.value[row].steps[col]
}

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
const steps = ref(
  allSamples.value.slice(0, rows).map(sample => ({
    name: sample.name,
    steps: Array(cols.value).fill(false),
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

function triggerMelodyNote(note, time) {
  if (!melodySynth) return

  if (selectedSynth.value === 'PianoSampler') {
    melodySynth.triggerAttack(note, time)
  } else {
    melodySynth.triggerAttackRelease(note, '8n', time)
  }
}


function scheduleTransport() {
  Tone.Transport.scheduleRepeat((time) => {
    currentStep.value = stepIndex.value

    // Drums
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

    // Vocals (only trigger once per loop)
    if (vocalPlayer && vocalReady && stepIndex.value === 0) {
      vocalPlayer.start(time)
    }

    // Melody notes
    melodySteps.value.forEach((row) => {
      if (row.steps[stepIndex.value]) {
        triggerMelodyNote(row.note, time)
      }
    })


    // Move to next step
    stepIndex.value = (stepIndex.value + 1) % cols.value
  }, '8n')
}


const sampler = new Tone.Sampler({
  C4: '/samples/piano/C4.wav',
  D4: '/samples/piano/D4.wav',
}).toDestination()


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
    steps: Array(cols.value).fill(false),
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

import { watch } from 'vue'

watch(cols, (newColCount) => {
  // Resize drums
  steps.value.forEach(row => {
    const len = row.steps.length
    if (newColCount > len) {
      row.steps.push(...Array(newColCount - len).fill(false))
    } else if (newColCount < len) {
      row.steps.splice(newColCount)
    }
  })

  // Resize melody
  melodySteps.value.forEach(row => {
    const len = row.steps.length
    if (newColCount > len) {
      row.steps.push(...Array(newColCount - len).fill(false))
    } else if (newColCount < len) {
      row.steps.splice(newColCount)
    }
  })
})




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

.melody-controls {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 0 0px 0px;
  font-size: 14px;
  /* color: #ccc; */
}

.melody-controls select {
  background: #2c2c2c;
  color: rgb(186, 186, 186);
  border: 0.5px solid #444;
  padding: 8px 50px;
  border-radius: 4px;
}
</style>
