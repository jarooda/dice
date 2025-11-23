<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import DiceBox from '@3d-dice/dice-box-threejs'

type DiceType = 4 | 6 | 8 | 10 | 12 | 20 | 100

const selectedDice = ref<DiceType>(6)
const numberOfDice = ref<number>(1)
const rollResults = ref<number[]>([])
const isRolling = ref<boolean>(false)
let diceBox: any = null

const totalResult = computed(() => {
  return rollResults.value.reduce((sum, value) => sum + value, 0)
})

const resultDisplay = computed(() => {
  if (rollResults.value.length === 0) return ''
  if (rollResults.value.length === 1) return rollResults.value[0].toString()
  return `${rollResults.value.join(' + ')} = ${totalResult.value}`
})

onMounted(async () => {
  diceBox = new DiceBox('#dice-box', {
    sounds: false,
    theme_colorset: 'white',
    theme_material: 'plastic',
    onRollComplete: (results: any) => {
      // Extract rolls from the first set
      const rolls = results.sets[0].rolls

      // Get the values directly - d100 already shows 10, 20, 30...100
      rollResults.value = rolls.map((r: any) => r.value)
      isRolling.value = false
    },
  })

  await diceBox.initialize()
})

const rollDice = () => {
  if (!diceBox || isRolling.value) return

  isRolling.value = true
  rollResults.value = []

  const diceType = `d${selectedDice.value}`
  const diceNotation = `${numberOfDice.value}${diceType}`

  diceBox.roll(diceNotation)
}

const getDiceName = (dice: DiceType): string => {
  return dice === 100 ? 'D%' : `D${dice}`
}

const dices = [
  { value: 4, label: 'D4' },
  { value: 6, label: 'D6' },
  { value: 8, label: 'D8' },
  { value: 10, label: 'D10' },
  { value: 12, label: 'D12' },
  { value: 20, label: 'D20' },
  { value: 100, label: 'D%' },
]
</script>

<template>
  <div class="app-container">
    <div class="controls-section">
      <h1 class="title">Dice Roller</h1>

      <div class="control-group">
        <h2 class="control-title">Select Dice Type</h2>
        <div class="dice-selector">
          <label
            v-for="dice in dices"
            :key="dice.value"
            class="dice-option"
            :class="{ active: selectedDice === dice.value }"
          >
            <input type="radio" :value="dice.value" v-model="selectedDice" />
            <span class="dice-label">{{ dice.label }}</span>
          </label>
        </div>
      </div>

      <div class="control-group">
        <h2 class="control-title">Number of Dice</h2>
        <div class="number-selector">
          <label
            v-for="n in 5"
            :key="n"
            class="number-option"
            :class="{ active: numberOfDice === n }"
          >
            <input type="radio" :value="n" v-model.number="numberOfDice" />
            <span class="number-label">{{ n }}</span>
          </label>
        </div>
      </div>

      <button class="roll-button" @click="rollDice" :disabled="isRolling">
        <span v-if="isRolling">🎲 Rolling...</span>
        <span v-else>🎲 Roll {{ numberOfDice }}{{ getDiceName(selectedDice) }}</span>
      </button>
    </div>

    <div class="results-section">
      <span class="results-title">Results:</span>
      <span class="results-display">{{ resultDisplay || '-' }}</span>
    </div>
    <!-- 3D Dice Box Container -->
    <div id="dice-box"></div>
  </div>
</template>

<style scoped>
* {
  box-sizing: border-box;
}

.app-container {
  height: 100vh;
  background: #f7fafc;
  padding: 0.5rem;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  overflow: hidden;
}

.controls-section {
  background: #ffffff;
  border-radius: 0.5rem;
  padding: 1rem;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  flex-shrink: 0;
}

.title {
  font-size: 1.25rem;
  font-weight: 700;
  text-align: center;
  margin: 0 0 0.75rem 0;
  color: #1a202c;
}

.control-group {
  margin-bottom: 0.75rem;
}

.control-title {
  font-size: 0.875rem;
  font-weight: 600;
  color: #4a5568;
  margin: 0 0 0.5rem 0;
}

.dice-selector {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(60px, 1fr));
  gap: 0.5rem;
}

.dice-option {
  position: relative;
  cursor: pointer;
  user-select: none;
}

.dice-option input[type='radio'] {
  position: absolute;
  opacity: 0;
  width: 0;
  height: 0;
}

.dice-label {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0.5rem;
  background: #f7fafc;
  border: 2px solid #e2e8f0;
  border-radius: 0.375rem;
  font-weight: 600;
  font-size: 0.875rem;
  color: #2d3748;
  transition: all 0.15s ease;
  min-height: 40px;
}

.dice-option:hover .dice-label {
  background: #edf2f7;
  border-color: #cbd5e0;
}

.dice-option.active .dice-label {
  background: #4299e1;
  color: white;
  border-color: #4299e1;
}

.number-selector {
  display: flex;
  gap: 0.5rem;
  justify-content: center;
}

.number-option {
  position: relative;
  cursor: pointer;
  user-select: none;
}

.number-option input[type='radio'] {
  position: absolute;
  opacity: 0;
  width: 0;
  height: 0;
}

.number-label {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 40px;
  height: 40px;
  background: #f7fafc;
  border: 2px solid #e2e8f0;
  border-radius: 0.375rem;
  font-weight: 600;
  font-size: 1rem;
  color: #2d3748;
  transition: all 0.15s ease;
}

.number-option:hover .number-label {
  background: #edf2f7;
  border-color: #cbd5e0;
}

.number-option.active .number-label {
  background: #4299e1;
  color: white;
  border-color: #4299e1;
}

.roll-button {
  width: 100%;
  padding: 0.75rem;
  font-size: 1rem;
  font-weight: 600;
  background: #4299e1;
  color: white;
  border: none;
  border-radius: 0.375rem;
  cursor: pointer;
  transition: all 0.15s ease;
}

.roll-button:hover:not(:disabled) {
  background: #3182ce;
}

.roll-button:active:not(:disabled) {
  background: #2c5282;
}

.roll-button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.results-section {
  padding: 0.75rem;
  background: #ffffff;
  border-radius: 0.375rem;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  flex-shrink: 0;
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.results-title {
  font-size: 0.875rem;
  font-weight: 600;
  color: #4a5568;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.results-display {
  font-size: 1.5rem;
  font-weight: 700;
  color: #4299e1;
  font-family: 'Courier New', monospace;
}

#dice-box {
  flex: 1;
  width: 100%;
  min-height: 0;
  background: #ffffff;
  border-radius: 0.5rem;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

/* Mobile optimizations */
@media (max-width: 640px) {
  .app-container {
    padding: 0.5rem;
  }

  .controls-section {
    padding: 0.75rem;
  }

  .dice-selector {
    grid-template-columns: repeat(auto-fit, minmax(50px, 1fr));
    gap: 0.375rem;
  }

  .dice-label {
    padding: 0.375rem;
    font-size: 0.8125rem;
    min-height: 36px;
  }

  .number-label {
    width: 36px;
    height: 36px;
    font-size: 0.875rem;
  }

  .roll-button {
    padding: 0.625rem;
    font-size: 0.9375rem;
  }

  #dice-box {
    min-height: 200px;
  }
}

/* Landscape mobile */
@media (max-width: 900px) and (orientation: landscape) {
  .app-container {
    flex-direction: row;
  }

  .controls-section {
    flex: 0 0 250px;
    max-height: calc(100vh - 1rem);
    overflow-y: auto;
  }

  #dice-box {
    flex: 1;
    min-height: auto;
    height: calc(100vh - 1rem);
  }
}

/* Touch device optimizations */
@media (hover: none) and (pointer: coarse) {
  .dice-option:hover .dice-label,
  .number-option:hover .number-label {
    transform: none;
  }

  .dice-option:active .dice-label {
    transform: scale(0.95);
  }

  .number-option:active .number-label {
    transform: scale(0.95);
  }
}
</style>
