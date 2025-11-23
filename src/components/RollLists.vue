<script setup lang="ts">
import HistoryIcon from './icons/History.vue'

interface RollHistory {
  type: string
  rolls: number[]
}

interface Props {
  history: RollHistory[]
}

defineProps<Props>()
const emit = defineEmits<{
  clearHistory: []
}>()

const getTotal = (rolls: number[]) => {
  return rolls.reduce((sum, value) => sum + value, 0)
}

const formatRoll = (roll: RollHistory) => {
  if (roll.rolls.length === 1) {
    return roll.rolls[0]!.toString()
  }
  return `${roll.rolls.join(' + ')} = ${getTotal(roll.rolls)}`
}

const clearHistory = () => {
  emit('clearHistory')
}
</script>

<template>
  <div class="roll-history">
    <div v-if="history.length === 0" class="empty-state">
      <HistoryIcon class="empty-icon" />
      <p class="empty-text">No roll history yet</p>
      <p class="empty-subtext">Start rolling dice to see your history here</p>
    </div>
    <div v-else class="history-list">
      <div v-for="(roll, index) in history" :key="index" class="history-item">
        <div class="roll-type">{{ roll.type }}</div>
        <div class="roll-result">{{ formatRoll(roll) }}</div>
      </div>
      <button class="clear-history-button" @click="clearHistory">Clear History</button>
    </div>
  </div>
</template>
