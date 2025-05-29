<template>
    <div
        class="champion-card"
        :class="{ selected }"
        @click="$emit('toggle-selection', champion.id)">
      <img :src="imageUrl" :alt="champion.name" />
      <p>{{ champion.name }}</p>
      <small v-if="selected" class="date">Joué le {{  formatedDate }}</small>
    </div>
</template>
  
<script setup>
import { computed } from 'vue'

const props = defineProps({
    champion: Object,
    version: String,
    selected: Boolean,
    selectionDate: String
})
  
const imageUrl = computed(() => {
    return `https://ddragon.leagueoflegends.com/cdn/${props.version}/img/champion/${props.champion.image.full}`
})

const formatedDate = computed(() => {
  if (!props.selectionDate) return ''
  const date = new Date(props.selectionDate)
  const day = String(date.getDate()).padStart(2, '0')
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const year = date.getFullYear()
  return `${day}/${month}/${year}`
})
</script>
  
<style scoped>
.champion-card {
    width: 120px;
    padding: 10px;
    text-align: center;
    border: solid 2px #213547;
    border-radius: 10px;
    font-weight: bold;
    background-color: #0A0A0A;
}
  
.champion-card img {
    width: 100%;
    border-radius: 8px;
    box-shadow: 0 2px 6px rgba(0,0,0,0.2);
}

.champion-card.selected {
  background-color: #4caf50;
  color: white;
}

.date {
  font-size: 0.75rem;
  color: white;
  opacity: 0.9;
}
</style>
  