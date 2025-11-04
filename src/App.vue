<script setup lang="ts">
import { ref ,onMounted, onUnmounted } from 'vue'

defineProps<{ 
  time: string,
  author: string,
  crypto: string,
  cryptoIcon: string,
  cryptoPrice: number,
  weather: string,
  weatherIcon: string 
}>()

const time = ref("TIME HERE")
const author = ref("")
const crypto = ref("")
const cryptoIcon = ref("")
const cryptoPrice = ref(0)
const weather = ref("")
const weatherIcon = ref("")

// unsplash background image
fetch("https://apis.scrimba.com/unsplash/photos/random?orientation=landscape&query=nature")
  .then(res => res.json())
  .then(data => {
    document.body.style.backgroundImage = `url(${data.urls.full})`
    author.value = data.user.name
    console.log(data.urls.full)
  })
  .catch(error => {
    console.error(error)
    // default image if an error happens
    document.body.style.backgroundImage = `url(https://images.unsplash.com/photo-1507525428034-b723cf961d3e?crop=entropy&cs=srgb&fm=jpg&ixid=M3wxNDI0NzB8MHwxfHJhbmRvbXx8fHx8fHx8fDE3NjE5NTY3MzJ8&ixlib=rb-4.1.0&q=85)`
  })

// crypto price
fetch("https://api.coingecko.com/api/v3/coins/bitcoin")
    .then(res => res.json())
    .then(data => {
      console.log(data)
      crypto.value = data.name
      cryptoIcon.value = data.image.small
      cryptoPrice.value = data.market_data.current_price.aud
    })

// format currency
function currency(number: number) {
  return new Intl.NumberFormat('en-AU', { style: 'currency', currency: 'AUD', minimumFractionDigits: 0 }).format(number)
}


// time
let timer: number | undefined

onMounted(() => {
  updateClock()
  timer = setInterval(updateClock, 1000)
})

onUnmounted(() => {
  clearInterval(timer)
})

const updateClock = () => {
  time.value = new Date().toLocaleTimeString("en-AU", {
    timeStyle: 'short'
  })
}

// weather
if (navigator.geolocation) {
  navigator.geolocation.getCurrentPosition(
      async ({coords})=> {
        const locWeather = await getWeather(coords)
        console.log(locWeather)
        weatherIcon.value = `https://openweathermap.org/img/wn/${locWeather.weather[0].icon}.png`
        weather.value = `${locWeather.name} ${Math.round(locWeather.main.temp)}˚C`
      }, 
      err => console.log(err)
  )
}

async function getWeather({latitude, longitude}: {latitude: number, longitude: number}) {
  //const url = `https://api.openweathermap.org/data/2.5/weather?lat=${latitude}&lon=${longitude}&units=metric&appid=${apiKey}`;
  // using openweather api through scrimba, no need for api key
  const url = `https://apis.scrimba.com/openweathermap/data/2.5/weather?lat=${latitude}&lon=${longitude}&units=metric`;
  const res = await fetch(url)
  return await res.json()
}

</script>

<template>
 <main>
  <div class="top">
    <div class="crypto">
      <img :src="cryptoIcon" :alt="crypto" /> 
      <div class="crypto-info">
        <p>{{ crypto }}</p>
        <p>{{ currency(cryptoPrice) }} AUD</p>
      </div>
    </div>
    <div class="weather">
      <img :src="weatherIcon" />
      <p>{{ weather }}</p>
    </div>
  </div>
  <h1>{{ time }}</h1>
  <p>{{ author }}</p>
 </main> 
</template>

<style scoped>
main {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  height: 100vh;
  width: 100%;
  padding: 1rem;
}

h1 {
  align-self: center;
}

.top {
  display: flex;
  justify-content: space-between;
}

.crypto {
  display: flex;
  align-items: center;
  gap: 1em;
}

.crypto-info p {
  margin: 0.5rem;
}

.weather {
  text-align: center;
}

.weather img {
    width: 50px;
    height: 30px;
    object-fit: cover;
}

.weather p {
  margin: 0;
}
</style>
