<template>
  <div class="magic-pipe-container">
    <h1>Magic Pipe Water System</h1>
    
    <!-- Mathematical Analysis -->
    <div class="analysis-panel">
      <h2>Mathematical Analysis</h2>
      <div class="calculation">
        <p><strong>Magic Pipe Multiplier:</strong> {{ multiplier }}x</p>
        <p><strong>Input:</strong> 1L → <strong>Output:</strong> {{ multiplier }}L</p>
        <p><strong>Requirements:</strong> Permanent watering (1L/min) + Permanent filling</p>
        <p><strong>Minimum Jugs Required:</strong> {{ minJugsRequired }} (3 full + 2 empty initially)</p>
        <p><strong>Net Water Gain per Cycle:</strong> {{ (multiplier - 1).toFixed(2) }}L</p>
        <p><strong>Operation:</strong> Both watering and filling run simultaneously forever</p>
      </div>
    </div>

    <!-- Controls -->
    <div class="controls">
      <button @click="toggleSimulation" :class="{ active: isRunning }">
        {{ isRunning ? 'Stop' : 'Start' }} Simulation
      </button>
      <button @click="resetSimulation">Reset</button>
      <div class="speed-control">
        <label>Speed: </label>
        <input type="range" v-model="simulationSpeed" min="0.1" max="3" step="0.1">
        <span>{{ simulationSpeed }}x</span>
      </div>
    </div>

    <!-- Statistics -->
    <div class="stats">
      <div class="stat">
        <label>Time:</label>
        <span>{{ formatTime(currentTime) }}</span>
      </div>
      <div class="stat">
        <label>Total Water Used:</label>
        <span>{{ totalWaterUsed.toFixed(2) }}L</span>
      </div>
      <div class="stat">
        <label>Total Water Generated:</label>
        <span>{{ totalWaterGenerated.toFixed(2) }}L</span>
      </div>
    </div>

    <!-- Jug Visualization -->
    <div class="jug-system">
      <div class="flower-section">
        <h3>🌸 Flower Watering</h3>
        <div class="flower-display">
          <div class="flower" v-for="n in 5" :key="n">🌷</div>
        </div>
        <div class="water-flow" :class="{ active: isWateringFlowers }">
          Water flowing: {{ isWateringFlowers ? '1L/min' : '0L/min' }}
        </div>
      </div>

      <div class="magic-pipe-section">
        <h3>✨ Magic Pipe</h3>
        <div class="pipe-visual">
          <div class="pipe-input">1L</div>
          <div class="pipe" :class="{ active: isPipeActive }">
            <div class="pipe-flow"></div>
            ✨ {{ multiplier }}x ✨
          </div>
          <div class="pipe-output">{{ multiplier }}L</div>
        </div>
      </div>

      <div class="jugs-section">
        <h3>🏺 Water Jugs</h3>
        <div class="jugs-grid">
          <div 
            v-for="(jug, index) in jugs" 
            :key="index" 
            class="jug"
            :class="jug.state"
          >
            <div class="jug-container">
              <div class="jug-water" :style="{ height: (jug.volume / 1.15 * 100) + '%' }"></div>
              <div class="jug-label">
                Jug {{ index + 1 }}<br>
                {{ jug.volume.toFixed(2) }}L<br>
                <small>{{ jug.state }}</small>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

// Configuration
const multiplier = ref(1.15)
const simulationSpeed = ref(1)
const flowRate = 1 // L/minute

// Simulation state
const isRunning = ref(false)
const currentTime = ref(0)
const totalWaterUsed = ref(0)
const totalWaterGenerated = ref(0)

// Jug states: 'empty', 'filling', 'full', 'emptying', 'watering'
const jugs = ref([])

// Simulation tracking
const isWateringFlowers = ref(false)
const isPipeActive = ref(false)
let simulationInterval = null

// Computed properties
const minJugsRequired = computed(() => {
  // For eternal operation with both watering and filling simultaneously:
  // - Time to empty 1.15L jug for flowers: 1.15 minutes
  // - Time to fill 1L through pipe to get 1.15L: 1 minute
  // - We need: 1 watering + 1 filling + 1 full ready + 1 empty ready + buffer
  // Minimum = 5 jugs to ensure continuous operation
  return 5
})

// Initialize jugs
const initializeJugs = () => {
  jugs.value = []
  const totalJugs = minJugsRequired.value
  
  for (let i = 0; i < totalJugs; i++) {
    jugs.value.push({
      volume: i < 3 ? multiplier.value : 0, // Start with 3 full jugs
      state: i < 3 ? 'full' : 'empty',
      timeInState: 0
    })
  }
}

// Simulation logic
const updateSimulation = () => {
  const deltaTime = 0.1 * simulationSpeed.value // 100ms intervals
  currentTime.value += deltaTime
  
  // Update each jug
  jugs.value.forEach((jug, index) => {
    jug.timeInState += deltaTime
    
    switch (jug.state) {
      case 'filling':
        jug.volume += flowRate * deltaTime
        if (jug.volume >= multiplier.value) {
          jug.volume = multiplier.value
          jug.state = 'full'
          jug.timeInState = 0
          totalWaterGenerated.value += multiplier.value - 1 // Net gain
        }
        break
        
      case 'emptying':
        jug.volume -= flowRate * deltaTime
        if (jug.volume <= 0) {
          jug.volume = 0
          jug.state = 'empty'
          jug.timeInState = 0
        }
        break
        
      case 'watering':
        jug.volume -= flowRate * deltaTime
        totalWaterUsed.value += flowRate * deltaTime
        if (jug.volume <= 0) {
          jug.volume = 0
          jug.state = 'empty'
          jug.timeInState = 0
          isWateringFlowers.value = false
        }
        break
    }
  })
  
  // Manage jug assignments
  manageJugOperations()
}

const manageJugOperations = () => {
  const fullJugs = jugs.value.filter(jug => jug.state === 'full')
  const emptyJugs = jugs.value.filter(jug => jug.state === 'empty')
  const fillingJugs = jugs.value.filter(jug => jug.state === 'filling')
  const wateringJugs = jugs.value.filter(jug => jug.state === 'watering')
  const emptyingJugs = jugs.value.filter(jug => jug.state === 'emptying')
  
  // PERMANENT WATERING: Always ensure flowers are being watered
  if (wateringJugs.length === 0 && fullJugs.length > 0) {
    fullJugs[0].state = 'watering'
    fullJugs[0].timeInState = 0
    isWateringFlowers.value = true
  }
  
  // PERMANENT FILLING: Always try to fill empty jugs if we have resources
  // We need at least 2 full jugs: 1 for watering, 1 for pipe source
  const availableFullJugs = fullJugs.length + (wateringJugs.length > 0 ? 0 : 1)
  
  if (fillingJugs.length === 0 && emptyJugs.length > 0 && availableFullJugs >= 2) {
    // Find a full jug that's not being used for watering
    const sourceJug = fullJugs.find(jug => jug.state === 'full')
    const targetJug = emptyJugs[0]
    
    if (sourceJug && targetJug) {
      sourceJug.state = 'emptying'
      sourceJug.timeInState = 0
      targetJug.state = 'filling'
      targetJug.timeInState = 0
      isPipeActive.value = true
    }
  }
  
  // Update pipe active status
  isPipeActive.value = fillingJugs.length > 0 || emptyingJugs.length > 0
  
  // Update flower watering status
  isWateringFlowers.value = wateringJugs.length > 0
}

// Control functions
const toggleSimulation = () => {
  isRunning.value = !isRunning.value
  
  if (isRunning.value) {
    simulationInterval = setInterval(updateSimulation, 100)
  } else {
    clearInterval(simulationInterval)
  }
}

const resetSimulation = () => {
  isRunning.value = false
  clearInterval(simulationInterval)
  currentTime.value = 0
  totalWaterUsed.value = 0
  totalWaterGenerated.value = 0
  isWateringFlowers.value = false
  isPipeActive.value = false
  initializeJugs()
}

const formatTime = (time) => {
  const minutes = Math.floor(time)
  const seconds = Math.floor((time % 1) * 60)
  return `${minutes}:${seconds.toString().padStart(2, '0')}`
}

// Lifecycle
onMounted(() => {
  initializeJugs()
})

onUnmounted(() => {
  if (simulationInterval) {
    clearInterval(simulationInterval)
  }
})
</script>

<style scoped>
.magic-pipe-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Arial', sans-serif;
}

h1 {
  text-align: center;
  color: #2c3e50;
  margin-bottom: 30px;
}

.analysis-panel {
  background: #f8f9fa;
  padding: 20px;
  border-radius: 10px;
  margin-bottom: 20px;
}

.analysis-panel h2 {
  color: #34495e;
  margin-bottom: 15px;
}

.calculation {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 15px;
}

.calculation p {
  background: white;
  padding: 10px;
  border-radius: 5px;
  margin: 0;
  border-left: 4px solid #3498db;
}

.controls {
  display: flex;
  gap: 20px;
  align-items: center;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

button {
  background: #3498db;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
  transition: background 0.3s;
}

button:hover {
  background: #2980b9;
}

button.active {
  background: #e74c3c;
}

.speed-control {
  display: flex;
  align-items: center;
  gap: 10px;
}

.stats {
  display: flex;
  gap: 30px;
  margin-bottom: 30px;
  flex-wrap: wrap;
}

.stat {
  background: #ecf0f1;
  padding: 15px;
  border-radius: 8px;
  min-width: 150px;
}

.stat label {
  font-weight: bold;
  color: #2c3e50;
}

.stat span {
  display: block;
  font-size: 1.2em;
  color: #27ae60;
  margin-top: 5px;
}

.jug-system {
  display: grid;
  grid-template-columns: 1fr 1fr 2fr;
  gap: 30px;
  margin-top: 30px;
}

.flower-section, .magic-pipe-section, .jugs-section {
  background: white;
  border: 2px solid #bdc3c7;
  border-radius: 10px;
  padding: 20px;
}

.flower-display {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin: 20px 0;
}

.flower {
  font-size: 2em;
  animation: sway 2s ease-in-out infinite alternate;
}

@keyframes sway {
  0% { transform: rotate(-5deg); }
  100% { transform: rotate(5deg); }
}

.water-flow {
  text-align: center;
  padding: 10px;
  background: #ecf0f1;
  border-radius: 5px;
  transition: background 0.3s;
}

.water-flow.active {
  background: #3498db;
  color: white;
}

.pipe-visual {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 20px;
  margin: 20px 0;
}

.pipe {
  background: linear-gradient(45deg, #f39c12, #e67e22);
  color: white;
  padding: 15px 30px;
  border-radius: 20px;
  font-weight: bold;
  position: relative;
  overflow: hidden;
}

.pipe.active::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255,255,255,0.4), transparent);
  animation: shimmer 1s infinite;
}

@keyframes shimmer {
  0% { left: -100%; }
  100% { left: 100%; }
}

.pipe-input, .pipe-output {
  background: #3498db;
  color: white;
  padding: 10px 15px;
  border-radius: 10px;
  font-weight: bold;
}

.jugs-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  gap: 20px;
}

.jug {
  text-align: center;
}

.jug-container {
  width: 80px;
  height: 120px;
  background: #ecf0f1;
  border: 3px solid #95a5a6;
  border-radius: 0 0 20px 20px;
  position: relative;
  margin: 0 auto 10px;
  overflow: hidden;
}

.jug-water {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  background: linear-gradient(to top, #3498db, #5dade2);
  transition: height 0.3s ease;
  border-radius: 0 0 17px 17px;
}

.jug.filling .jug-water {
  background: linear-gradient(to top, #f39c12, #f7dc6f);
}

.jug.watering .jug-water {
  background: linear-gradient(to top, #e74c3c, #ec7063);
}

.jug.emptying .jug-water {
  background: linear-gradient(to top, #9b59b6, #bb8fce);
}

.jug-label {
  font-size: 12px;
  color: #2c3e50;
  line-height: 1.2;
}

.jug-label small {
  text-transform: uppercase;
  font-weight: bold;
  color: #7f8c8d;
}

h3 {
  color: #2c3e50;
  margin-bottom: 15px;
  text-align: center;
}

@media (max-width: 768px) {
  .jug-system {
    grid-template-columns: 1fr;
  }
  
  .controls {
    justify-content: center;
  }
  
  .stats {
    justify-content: center;
  }
}
</style>