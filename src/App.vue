<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import DiceBox from '@3d-dice/dice-box-threejs'
import { useStorage } from '@vueuse/core'
import { sample } from 'jalutils'
import confetti from 'canvas-confetti'
import RollLists from './components/RollLists.vue'
import HistoryIcon from './components/icons/History.vue'
import './styles/main.css'

type DiceType = 4 | 6 | 8 | 10 | 12 | 20 | 100

interface RollHistory {
  type: string
  rolls: number[]
}

const selectedDice = ref<DiceType>(20)
const numberOfDice = ref<number>(1)
const rollResults = ref<number[]>([])
const isRolling = ref<boolean>(false)
const rollHistory = useStorage<RollHistory[]>('dice-roll-history', [])
const maxHistoryItems = 10
const showHistoryModal = ref<boolean>(false)
let diceBox: any = null

const totalResult = computed(() => {
  return rollResults.value.reduce((sum, value) => sum + value, 0)
})

const resultDisplay = computed(() => {
  if (rollResults.value.length === 0) return ''
  if (rollResults.value.length === 1) return rollResults.value[0]!.toString()
  return `${rollResults.value.join(' + ')} = ${totalResult.value}`
})

const resultSuffix = computed(() => {
  if (rollResults.value.length === 0) return ''
  let suffix = ''
  if (selectedDice.value === 100) {
    suffix += '%'
  }
  return suffix
})

const getColorSet = (): string => {
  const sets = ['acid', 'fire', 'ice', 'dragon', 'radiant', 'poison']
  return sample(sets)!
}

onMounted(async () => {
  diceBox = new DiceBox('#dice-box', {
    sounds: false,
    theme_texture: 'wood',
    theme_colorset: getColorSet(),
    theme_material: 'plastic',
    onRollComplete: (results: any) => {
      // Extract rolls from the first set
      const rolls = results.sets[0].rolls

      // Get the values directly - d100 already shows 10, 20, 30...100
      rollResults.value = rolls.map((r: any) => r.value)

      // Check for max roll and trigger confetti celebration
      const maxPossible = numberOfDice.value * selectedDice.value
      const total = rollResults.value.reduce((sum, value) => sum + value, 0)
      if (total === maxPossible) {
        // Create 🎉 emoji shape
        const scalar = 2
        const party = confetti.shapeFromText({ text: '🎉', scalar })

        // Fire confetti from multiple angles
        const duration = 3000
        const animationEnd = Date.now() + duration
        const defaults = {
          startVelocity: 30,
          spread: 360,
          ticks: 60,
          zIndex: 9999,
          shapes: [party],
          scalar,
        }

        const randomInRange = (min: number, max: number) => {
          return Math.random() * (max - min) + min
        }

        const interval = setInterval(() => {
          const timeLeft = animationEnd - Date.now()

          if (timeLeft <= 0) {
            isRolling.value = false
            return clearInterval(interval)
          }

          const particleCount = 50 * (timeLeft / duration)

          // Fire confetti from left side
          confetti({
            ...defaults,
            particleCount,
            origin: { x: randomInRange(0.1, 0.3), y: Math.random() - 0.2 },
          })

          // Fire confetti from right side
          confetti({
            ...defaults,
            particleCount,
            origin: { x: randomInRange(0.7, 0.9), y: Math.random() - 0.2 },
          })
        }, 250)
      } else {
        isRolling.value = false
      }

      // Save to history
      const diceType = `${numberOfDice.value}d${selectedDice.value}`
      const newRoll: RollHistory = {
        type: diceType,
        rolls: rollResults.value,
      }

      // Add to beginning of array and limit to maxHistoryItems
      rollHistory.value = [newRoll, ...rollHistory.value].slice(0, maxHistoryItems)
    },
  })

  await diceBox.initialize()
})

const rollDice = () => {
  if (!diceBox || isRolling.value) return

  isRolling.value = true
  rollResults.value = []
  const colorset = getColorSet()

  // Randomize color set on every roll
  diceBox.updateConfig({
    theme_colorset: colorset,
  })

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

const toggleHistoryModal = () => {
  showHistoryModal.value = !showHistoryModal.value
}

const clearRollHistory = () => {
  rollHistory.value = []
}
</script>

<template>
  <div class="app-container">
    <div class="controls-section">
      <div class="header">
        <h1 class="title">Dice Roller</h1>
        <button
          class="history-button"
          @click="toggleHistoryModal"
          :class="{ active: rollHistory.length > 0 }"
          title="View Roll History"
        >
          <HistoryIcon />
          <span v-if="rollHistory.length > 0" class="history-count">{{ rollHistory.length }}</span>
        </button>
      </div>

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
      <span class="results-display">{{ resultDisplay || '-' }}{{ resultSuffix }}</span>
    </div>

    <!-- History Modal -->
    <Teleport to="body">
      <Transition name="modal">
        <div v-if="showHistoryModal" class="modal-overlay" @click="toggleHistoryModal">
          <div class="modal-content" @click.stop>
            <div class="modal-header">
              <h2 class="modal-title">Roll History</h2>
              <button class="modal-close" @click="toggleHistoryModal">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
                  <path
                    fill="currentColor"
                    d="M19 6.41L17.59 5L12 10.59L6.41 5L5 6.41L10.59 12L5 17.59L6.41 19L12 13.41L17.59 19L19 17.59L13.41 12z"
                  />
                </svg>
              </button>
            </div>
            <RollLists :history="rollHistory" @clear-history="clearRollHistory" />
          </div>
        </div>
      </Transition>
    </Teleport>

    <!-- 3D Dice Box Container -->
    <div id="dice-box"></div>
  </div>
</template>
