<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted, watch } from 'vue'

defineProps<{ msg: string }>()
/*
const players = ref<Map<string, number>>(new Map([

  ['PlayerA', 3],
  ['PlayerB', 10],
  ['PlayerC', 5],
  ['PlayerD', 4],
  ['PlayerE', 4],
  ['PlayerF', 3],
  ['PlayerG', 2],
  ['PlayerH', 1],
  ['PlayerI', 0],
  ['Player10', 0],
  ['Player11', 0],
  ['Player12', 0],
  ['Player13', 0],
  ['Player14', 1],
  ['Player15', 0],
  ['Player16', 0],
  ['Player17', 0],
  ['Player18', 0]


]))
    */
const LS_KEY = 'mrc_players'

const storedPlayers = localStorage.getItem(LS_KEY)
const players = ref<Map<string, number>>(storedPlayers
    ? new Map(JSON.parse(storedPlayers))
    : new Map([]))

watch(players, (val) => {
  localStorage.setItem(LS_KEY, JSON.stringify(Array.from(val.entries())))
}, { deep: true })

const newPlayerName = ref('')
const sorted = ref(false)
const hoveredPlayer = ref<string | null>(null)
const showTopThree = ref(false)


function resetPlayers() {
  players.value.clear()
  localStorage.removeItem(LS_KEY)
}
function updatePoints(name: string, newPoints: number) {
  players.value.set(name, newPoints)
  if (sorted.value) {
    sortByPoints()
  }
}
function addPlayer(name: string) {
  players.value.set(name, 0)
}

function deletePlayer(name: string) {
  players.value.delete(name)
}
function handleAddPlayer() {
  if(newPlayerName.value.trim()!== '') {
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

</script>

<template>
  <div class="top-bar">
    <h1>{{ msg }}</h1>

    <form @submit.prevent="handleAddPlayer">
      <input v-model="newPlayerName" placeholder="playername" id="playernameinput"/>
      <button type="submit" id="add">+</button>
    </form>

    <div>
      <label>
        <input type="checkbox" v-model="sorted" @change="handleSortChange" />
        Winners first
      </label>
      <br>
      <label>
        <input type="checkbox" v-model="showTopThree" @click="showTopThree = !showTopThree" />
        Ranks 1-3 only
      </label>
    </div>

  </div>

  <div class="players-grid">
  <div class="card" v-for="([name, points]) in visiblePlayers" :key="name">
    <button type="button"
            @click="updatePoints(name, points + 1)"
            @contextmenu.prevent="updatePoints(name, points - 1)"
            @mouseenter="hoveredPlayer = name"
            @mouseleave="hoveredPlayer = null"
    >{{ name }}<br>{{ points }}</button>
  </div>
  </div>

  <div class="footer">
      <button @click="resetPlayers" tabindex="-1">Reset</button>
  </div>

</template>
