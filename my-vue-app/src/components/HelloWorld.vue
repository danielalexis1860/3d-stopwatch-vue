```vue
<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

// Stopwatch state
const elapsedTime = ref(0)
const isRunning = ref(false)
const stopwatchCard = ref(null)

let startTime = 0
let timerInterval = null

// Format the stopwatch time
const formattedTime = () => {
  const total = elapsedTime.value

  const hours = Math.floor(total / 3600000)
  const minutes = Math.floor((total % 3600000) / 60000)
  const seconds = Math.floor((total % 60000) / 1000)
  const milliseconds = Math.floor((total % 1000) / 10)

  return {
    main: [
      hours.toString().padStart(2, '0'),
      minutes.toString().padStart(2, '0'),
      seconds.toString().padStart(2, '0'),
    ].join(':'),
    milli: '.' + milliseconds.toString().padStart(2, '0'),
  }
}

// Start the stopwatch
function startStopwatch() {
  if (isRunning.value) return

  startTime = Date.now() - elapsedTime.value

  timerInterval = setInterval(() => {
    elapsedTime.value = Date.now() - startTime
  }, 10)

  isRunning.value = true
}

// Pause or resume the stopwatch
function toggleStopwatch() {
  if (isRunning.value) {
    clearInterval(timerInterval)
    timerInterval = null
    isRunning.value = false
  } else {
    startStopwatch()
  }
}

// Reset the stopwatch
function resetStopwatch() {
  clearInterval(timerInterval)
  timerInterval = null
  elapsedTime.value = 0
  isRunning.value = false
}

// 3D mouse tilt effect
function handleMouseMove(event) {
  if (!stopwatchCard.value) return

  const xAxis = (window.innerWidth / 2 - event.clientX) / 25
  const yAxis = (window.innerHeight / 2 - event.clientY) / 25

  stopwatchCard.value.style.transition = 'none'
  stopwatchCard.value.style.transform =
    `rotateY(${xAxis}deg) rotateX(${yAxis}deg)`
}

// Reset the 3D tilt when the pointer leaves the page
function resetTilt() {
  if (!stopwatchCard.value) return

  stopwatchCard.value.style.transition = 'transform 0.5s ease-in-out'
  stopwatchCard.value.style.transform = 'rotateY(0deg) rotateX(0deg)'
}

onMounted(() => {
  window.addEventListener('mousemove', handleMouseMove)
  document.addEventListener('mouseleave', resetTilt)
})

onUnmounted(() => {
  clearInterval(timerInterval)
  window.removeEventListener('mousemove', handleMouseMove)
  document.removeEventListener('mouseleave', resetTilt)
})
</script>

<template>
  <main class="stopwatch-page">
    <!-- Ambient background lights -->
    <div class="ambient-glow glow-1"></div>
    <div class="ambient-glow glow-2"></div>

    <!-- 3D Stopwatch Card -->
    <section
      ref="stopwatchCard"
      class="stopwatch-container"
      :class="{ 'active-timer': isRunning }"
    >
      <div class="pulse-ring"></div>

      <h1 class="title">Chronometer 3D</h1>

      <!-- Digital time display -->
      <div class="display-panel">
        <div class="time-digits">
          {{ formattedTime().main }}
        </div>

        <div class="milliseconds">
          {{ formattedTime().milli }}
        </div>
      </div>

      <!-- Stopwatch controls -->
      <div class="controls">
        <button
          type="button"
          class="btn start-stop-btn"
          :class="{ running: isRunning }"
          @click="toggleStopwatch"
        >
          {{ isRunning ? 'PAUSE' : elapsedTime > 0 ? 'RESUME' : 'START' }}
        </button>

        <button
          type="button"
          class="btn reset-btn"
          @click="resetStopwatch"
        >
          RESET
        </button>
      </div>
    </section>
  </main>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;600;800;900&family=Inter:wght@300;400;600&display=swap');

:global(*) {
  box-sizing: border-box;
}

:global(html),
:global(body),
:global(#app) {
  margin: 0;
  min-height: 100%;
  width: 100%;
}

:global(body) {
  font-family: 'Inter', sans-serif;
  background: #0b0f19;
}

:global(button) {
  font: inherit;
}

.stopwatch-page {
  --bg-color: #0b0f19;
  --card-bg: rgba(255, 255, 255, 0.03);
  --card-border: rgba(255, 255, 255, 0.08);
  --primary-glow: #00f0ff;
  --secondary-glow: #7000ff;
  --accent-glow: #ff007b;
  --text-main: #ffffff;
  --text-muted: #94a3b8;

  position: relative;
  isolation: isolate;
  width: 100%;
  min-height: 100vh;
  min-height: 100dvh;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 30px 20px;
  overflow: hidden;
  perspective: 1000px;
  background-color: var(--bg-color);
  background-image:
    radial-gradient(
      circle at 10% 20%,
      rgba(112, 0, 255, 0.15) 0%,
      transparent 40%
    ),
    radial-gradient(
      circle at 90% 80%,
      rgba(0, 240, 255, 0.15) 0%,
      transparent 40%
    );
}

/* Ambient glowing spheres */
.ambient-glow {
  position: absolute;
  z-index: -1;
  width: 300px;
  height: 300px;
  border-radius: 50%;
  filter: blur(80px);
  opacity: 0.5;
  pointer-events: none;
  animation: floatGlow 8s ease-in-out infinite alternate;
}

.glow-1 {
  top: 10%;
  left: 15%;
  background: var(--secondary-glow);
}

.glow-2 {
  right: 15%;
  bottom: 10%;
  background: var(--primary-glow);
  animation-delay: -4s;
}

@keyframes floatGlow {
  from {
    transform: translateY(0) scale(1);
  }

  to {
    transform: translateY(-30px) scale(1.1);
  }
}

/* 3D glass card */
.stopwatch-container {
  position: relative;
  z-index: 1;
  width: 420px;
  max-width: 100%;
  padding: 40px 30px;
  text-align: center;
  background: var(--card-bg);
  border: 1px solid var(--card-border);
  border-radius: 30px;
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  box-shadow:
    0 30px 60px rgba(0, 0, 0, 0.5),
    inset 0 1px 0 rgba(255, 255, 255, 0.1);
  transform-style: preserve-3d;
  transition:
    transform 0.1s ease-out,
    box-shadow 0.3s ease;
}

.title {
  position: relative;
  transform: translateZ(30px);
  margin: 0 0 25px;
  color: var(--text-muted);
  font-family: 'Orbitron', sans-serif;
  font-size: 0.9rem;
  font-weight: 600;
  letter-spacing: 4px;
  text-transform: uppercase;
}

/* Digital display */
.display-panel {
  position: relative;
  overflow: hidden;
  transform: translateZ(50px);
  margin-bottom: 35px;
  padding: 25px 15px;
  background: rgba(0, 0, 0, 0.4);
  border: 1px solid rgba(255, 255, 255, 0.05);
  border-radius: 20px;
  box-shadow:
    inset 0 10px 20px rgba(0, 0, 0, 0.6),
    0 2px 5px rgba(255, 255, 255, 0.02);
}

.display-panel::after {
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  content: '';
  pointer-events: none;
  background: linear-gradient(
    135deg,
    transparent 0%,
    rgba(255, 255, 255, 0.03) 50%,
    transparent 100%
  );
}

.time-digits {
  color: var(--text-main);
  font-family: 'Orbitron', sans-serif;
  font-size: clamp(1.8rem, 7vw, 2.8rem);
  font-weight: 800;
  letter-spacing: 2px;
  white-space: nowrap;
  text-shadow: 0 0 20px rgba(0, 240, 255, 0.4);
  font-variant-numeric: tabular-nums;
}

.milliseconds {
  margin-top: 5px;
  color: var(--primary-glow);
  font-family: 'Orbitron', sans-serif;
  font-size: 1.2rem;
  font-weight: 600;
  text-shadow: 0 0 10px rgba(0, 240, 255, 0.6);
}

/* Buttons */
.controls {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 15px;
  transform: translateZ(40px);
}

.btn {
  position: relative;
  padding: 15px 8px;
  border: none;
  border-radius: 14px;
  font-family: 'Orbitron', sans-serif;
  font-size: 0.85rem;
  font-weight: 600;
  letter-spacing: 1px;
  cursor: pointer;
  outline: none;
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.3);
  transition: all 0.2s ease;
}

.btn:active {
  transform: translateY(3px) translateZ(-10px);
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
}

.btn:focus-visible {
  outline: 2px solid #ffffff;
  outline-offset: 4px;
}

/* Start, pause and resume */
.start-stop-btn {
  color: #0b0f19;
  background: linear-gradient(135deg, #00f0ff, #0077ff);
  box-shadow: 0 8px 20px rgba(0, 240, 255, 0.3);
}

.start-stop-btn:hover {
  filter: brightness(1.1);
  box-shadow: 0 12px 25px rgba(0, 240, 255, 0.5);
}

.start-stop-btn.running {
  color: #ffffff;
  background: linear-gradient(135deg, #ff007b, #7000ff);
  box-shadow: 0 8px 20px rgba(255, 0, 123, 0.3);
}

.start-stop-btn.running:hover {
  box-shadow: 0 12px 25px rgba(255, 0, 123, 0.5);
}

/* Reset */
.reset-btn {
  color: var(--text-muted);
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.reset-btn:hover {
  color: var(--text-main);
  background: rgba(255, 255, 255, 0.1);
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.4);
}

/* Active stopwatch glow */
.pulse-ring {
  position: absolute;
  z-index: -1;
  inset: -2px;
  border-radius: 32px;
  background: linear-gradient(
    135deg,
    var(--primary-glow),
    var(--secondary-glow)
  );
  opacity: 0;
  filter: blur(8px);
  transition: opacity 0.3s ease;
  pointer-events: none;
}

.stopwatch-container.active-timer .pulse-ring {
  opacity: 0.6;
}

.stopwatch-container.active-timer {
  box-shadow:
    0 30px 60px rgba(0, 0, 0, 0.5),
    0 0 30px rgba(0, 240, 255, 0.08),
    inset 0 1px 0 rgba(255, 255, 255, 0.1);
}

/* Responsive layout */
@media (max-width: 480px) {
  .stopwatch-page {
    padding: 20px 16px;
  }

  .stopwatch-container {
    padding: 32px 20px;
    border-radius: 24px;
  }

  .title {
    font-size: 0.75rem;
    letter-spacing: 3px;
  }

  .display-panel {
    padding: 24px 8px;
  }

  .controls {
    gap: 10px;
  }

  .btn {
    font-size: 0.75rem;
  }

  .ambient-glow {
    width: 200px;
    height: 200px;
  }
}

@media (prefers-reduced-motion: reduce) {
  .ambient-glow {
    animation: none;
  }

  .stopwatch-container,
  .btn,
  .pulse-ring {
    transition: none;
  }
}
</style>
```