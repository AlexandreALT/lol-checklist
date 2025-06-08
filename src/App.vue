<template>
  <div class="container">
    <div class="header">
      <h1>League of Legends champions</h1>

      <div class="search-container">
        <input type="text" v-model="search" placeholder="Search a champion..." class="searchbar" />
        <button v-if="search" class="clear-btn" @click="search = ''" aria-label="Clear search">&times;</button>
      </div>

      <div class="random-filters-counter">
        <div class="random">
          <button @click="openRandomChampionModal">Random</button>
        </div>

        <div class="filters">
          <button :class="{ active: filter === 'all' }" @click="filter = 'all'">All</button>
          <button :class="{ active: filter === 'selected' }" @click="filter = 'selected'">Played</button>
          <button :class="{ active: filter === 'unselected' }" @click="filter = 'unselected'">Not played</button>
        </div>

        <div class="progress-container">
          <svg class="progress-ring" width="60" height="60">
            <circle
              class="progress-ring__background"
              stroke="#ccc"
              stroke-width="6"
              fill="transparent"
              r="26"
              cx="30"
              cy="30"
            />
            <circle
              class="progress-ring__circle"
              :stroke-dasharray="circumference"
              :stroke-dashoffset="progressOffset"
              stroke="#42b983"
              stroke-width="6"
              fill="transparent"
              r="26"
              cx="30"
              cy="30"
            />
            <text
              x="50%"
              y="50%"
              dominant-baseline="middle"
              text-anchor="middle"
              font-size="10"
              font-weight="bold"
              fill="#F0EAD6"
            >
              {{ selectedCount }} / {{ totalCount }}
            </text>
          </svg>
        </div>
      </div>
    </div>

    <div class="grid">
      <ChampionCard
        v-for="champion in filteredChampions"
        :key="champion.id"
        :champion="champion"
        :version="version"
        :selected="champion.id in selectedChampions"
        :selectionDate="selectedChampions[champion.id]"
        @toggle-selection="toggleSelection"
      />
    </div>
  </div>

  <div v-if="showModal" class="modal" @click.self="closeModal">
    <div class="modal-content">
      <img
        v-if="randomChampion"
        :src="`https://ddragon.leagueoflegends.com/cdn/${version}/img/champion/${randomChampion.image.full}`"
        :alt="randomChampion.name"
        class="champion-image"
      />
      <h2 style="color: #F0EAD6">{{ randomChampion.name }}</h2>
      <button @click="closeModal">Close</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import ChampionCard from './components/ChampionCard.vue'

const champions = ref([])
const version = ref('')
const search = ref('')
const selectedChampions = ref({})
const filter = ref('all')

const selectedCount = computed(() => Object.keys(selectedChampions.value).length)
const totalCount = computed(() => champions.value.length)

const radius = 26
const circumference = 2 * Math.PI * radius

const progressOffset = computed(() => {
  const percent = totalCount.value > 0
    ? selectedCount.value / totalCount.value
    : 0
  return circumference * (1 - percent)
})

const showModal = ref(false)
const randomChampion = ref(null)

function openRandomChampionModal() {
  const unselected = champions.value.filter(champ => !(champ.id in selectedChampions.value))
  if (unselected.length === 0) {
    alert("You have already played each champions !")
    return
  }
  const index = Math.floor(Math.random() * unselected.length)
  randomChampion.value = unselected[index]
  showModal.value = true
}

function closeModal() {
  showModal.value = false
}


onMounted(() => {
  const saved = localStorage.getItem('selectedChampions')
  if (saved) {
    selectedChampions.value = JSON.parse(saved)
  }
})

const fetchChampions = async () => {
  try {
    const versionsRes = await fetch('https://ddragon.leagueoflegends.com/api/versions.json')
    const versions = await versionsRes.json()
    version.value = versions[0]

    const res = await fetch(`https://ddragon.leagueoflegends.com/cdn/${version.value}/data/fr_FR/champion.json`)
    const data = await res.json()

    champions.value = Object.values(data.data)
  } catch (error) {
    console.error('Erreur lors du chargement des champions :', error)
  }
}

const filteredChampions = computed(() => {
  const term = search.value.toLowerCase()
  let filtered = champions.value.filter(champ =>
    champ.name.toLowerCase().includes(term)
  )

  if (filter.value === 'selected') {
    filtered = filtered.filter(champ => champ.id in selectedChampions.value)
  } else if (filter.value === 'unselected') {
    filtered = filtered.filter(champ => !(champ.id in selectedChampions.value))
  }

  return filtered
})

watch(selectedChampions, (val) => {
  localStorage.setItem('selectedChampions', JSON.stringify(val))
}, { deep: true })

function toggleSelection(championId) {
  if (selectedChampions.value[championId]) {
    const confirmed = confirm("Confirm ?")
    if (confirmed) {
      const copy = { ...selectedChampions.value }
      delete copy[championId]
      selectedChampions.value = copy
    }
  } else {
    selectedChampions.value = {
      ...selectedChampions.value,
      [championId]: new Date().toISOString()
    }
  }
}

onMounted(fetchChampions)
</script>

<style>
.header {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  background-color: #3B1F63;
  z-index: 10;
  padding-bottom: 1rem;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  text-align: center;
}

.container {
  padding-top: 250px;
  max-width: 1000px;
  margin: auto;
  text-align: center;
}

.search-container {
  position: relative;
  max-width: 400px;
  width: 100%;
}

.searchbar {
  padding: 0.5rem 2.5rem 0.5rem 1rem;
  font-size: 1rem;
  width: 100%;
  border: 1px solid #ccc;
  border-radius: 8px;
  background-color: #F0EAD6;
  box-sizing: border-box;
}

.clear-btn {
  position: absolute;
  right: 0.75rem;
  top: 50%;
  transform: translateY(-50%);
  border: none;
  background: transparent;
  font-size: 1.3rem;
  cursor: pointer;
  color: #999;
  padding: 0;
  line-height: 1;
}

.clear-btn:hover {
  color: #555;
}

.grid {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 1.5rem;
  justify-items: center;
}

.random-filters-counter {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 1rem;
  width: 50%;
}

.filters {
  display: flex;
  gap: 10px;
}

.filters button,
.random button,
.modal button {
  margin: 0 5px;
  padding: 0.4rem 0.8rem;
  border: 1px solid #42b983;
  background: #F0EAD6;
  color: #42b983;
  border-radius: 4px;
  cursor: pointer;
  font-weight: 600;
  transition: background-color 0.2s ease;
}

.filters button.active,
.filters button:hover,
.random button:hover,
.modal button:hover {
  background-color: #42b983;
  color: #F0EAD6;
}

.progress-container {
  position: absolute;
  right: 0;
}

.progress-ring__background {
  stroke: #F0EAD6;
}

.progress-ring__circle {
  transition: stroke-dashoffset 0.35s;
  transform: rotate(-90deg);
  transform-origin: 50% 50%;
}

.random {
  position: absolute;
  left: 0;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
}

.modal-content {
  background: #3B1F63;
  color: #F0EAD6;
  padding: 2rem;
  border-radius: 10px;
  text-align: center;
  position: relative;
  max-width: 300px;
  width: 100%;
}

.champion-image {
  width: 120px;
  height: auto;
  border-radius: 8px;
  margin-bottom: 1rem;
}
</style>
