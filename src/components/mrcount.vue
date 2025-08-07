<script setup lang="ts">
import {ref, computed, onMounted, onUnmounted, watch} from 'vue'

defineProps<{ msg: string }>()

const newPlayerName = ref('')
const sorted = ref(false)
const hoveredPlayer = ref<string | null>(null)
const showTopThree = ref(false)
const LS_KEY = 'mrc_players'
const storedPlayers = localStorage.getItem(LS_KEY)
const players = ref<Map<string, number>>(storedPlayers
    ? new Map(JSON.parse(storedPlayers))
    : new Map([]))

watch(players, (val) => {
  localStorage.setItem(LS_KEY, JSON.stringify(Array.from(val.entries())))
}, {deep: true})


function resetPlayers() {
  if (!confirm('Are you sure you want to reset all players?')) {
    return
  }
  downloadPlayers()
  players.value.clear()
  localStorage.removeItem(LS_KEY)
}

function downloadPlayers() {
  let data = Array.from(players.value.entries())
      .map(([name, points]) => `${name},${points}`)
      .join('\n')
  const date = new Date().toLocaleDateString('de-DE', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit'
  })
  const header = 'MusikRatenCounter,' + date + '\nPlayer,Points\n'
  data = header + data
  const blob = new Blob([data], {type: 'text/plain'})
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = 'players.csv'
  a.click()
  URL.revokeObjectURL(url)
}

function updatePoints(name: string, newPoints: number) {
  players.value.set(name, newPoints)
  if (sorted.value) {
    sortByPoints()
  }
}

function addPlayer(name: string) {
  players.value.set(name, 1)
}

function deletePlayer(name: string) {
  players.value.delete(name)
}

function handleAddPlayer() {
  if (newPlayerName.value.trim() !== '') {
    if (players.value.has(newPlayerName.value.trim())) {
      alert('Player already exists!')
      return
    }
  } else {
    alert('Please enter a valid player name!')
    return
  }
  addPlayer(newPlayerName.value.trim())
  newPlayerName.value = ''
}

function sortByPoints() {
  const sortedPlayers = Array.from(players.value.entries()).sort((a, b) => b[1] - a[1])
  players.value = new Map(sortedPlayers)
}

function handleSortChange() {
  if (sorted.value) {
    sortByPoints()
  }
}

function handleKeydown(e: KeyboardEvent) {
  if (e.key === 'x' && hoveredPlayer.value) {
    deletePlayer(hoveredPlayer.value)
    hoveredPlayer.value = null
  }
}

onMounted(() => {
  window.addEventListener('keydown', handleKeydown)
})
onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown)
})

const visiblePlayers = computed(() => {
  const arr = Array.from(players.value.entries())
  if (showTopThree.value && arr.length > 0) {
    const sorted = arr.sort((a, b) => b[1] - a[1])
    const topPoints = [...new Set(sorted.map(([_, points]) => points))].slice(0, 3)
    return sorted.filter(([_, points]) => topPoints.includes(points))
  }
  return arr
})

const totalPoints = computed(() => {
  return Array.from(players.value.values()).reduce((sum, points) => sum + points, 0)
})


</script>

<template>
  <div class="top-bar">
    <div>
      <h1>{{ msg }}</h1>
      - {{ totalPoints }} points in total
    </div>

    <form @submit.prevent="handleAddPlayer">
      <input v-model="newPlayerName" placeholder="playername" id="playernameinput"/>
      <button type="submit" id="add">add</button>
    </form>

    <div>
      <label>
        <input type="checkbox" v-model="sorted" @change="handleSortChange"/>
        Winners first
      </label>
      <br>
      <label>
        <input type="checkbox" v-model="showTopThree" @click="showTopThree = !showTopThree"/>
        Ranks 1-3 only
      </label>
    </div>

  </div>

  <div class="players-grid">
    <div class="card" v-for="([name, points]) in visiblePlayers" :key="name">
      <div class="player-info"
           @mouseenter="hoveredPlayer = name"
           @mouseleave="hoveredPlayer = null"
      >
        {{ points }}<br>
        {{ name }}<br>
        <button type="button"
                @click="updatePoints(name, points - 1)"
        >-
        </button>
        <button type="button"
                @click="updatePoints(name, points + 1)"
        >+
        </button>
      </div>
    </div>
  </div>

  <div class="footer">
    <button @click="resetPlayers" tabindex="-1">reset?</button>
  </div>

</template>
