<script setup lang="ts">
import { ref } from 'vue'

defineProps<{ 
  time: string,
  author: string 
}>()

const time = ref("TIME HERE")
const author = ref("")


fetch("https://apis.scrimba.com/unsplash/photos/random?orientation=landscape&query=nature")
.then(res => res.json())
.then(data => {
	document.body.style.backgroundImage = `url(${data.urls.full})`
	author.value = data.user.name
  console.log(data.urls.full)
})

</script>

<template>
  <div class="top">
    <p>Crypto</p>
    <p>Weather</p>
  </div>
  <h1>{{ time }}</h1>
  <p>{{ author }}</p>
  
</template>

<style scoped>
.read-the-docs {
  color: #888;
}

.top {
  display: flex;
  justify-content: space-between;
}
</style>
