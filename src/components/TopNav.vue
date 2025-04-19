<template>
  <header class="top-nav">
    <h2 style="font-weight: 400; color: grey;">PULSE</h2>
    <div class="controls">
      <button class="metal-button" @click="$emit('toggle-transport')" title="Play / Pause">
        <span v-if="isPlaying">⏹</span>
        <span v-else>▶</span>
      </button>
    </div>
  </header>
</template>

<script setup>
defineProps({
  bpm: Number,
  isPlaying: Boolean
})
</script>

<style scoped>
.top-nav {
  grid-area: top-nav;
  background-color: #2c2c2c;
  padding: 0 1rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-bottom: 1px solid #444;
}

.controls button {
  color: #1e1e1e;
  padding: 0.6rem 0.8rem;
  border-radius: 6px;
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
</style>