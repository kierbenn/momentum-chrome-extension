<script setup lang="ts">
import { ref ,onMounted, onUnmounted } from 'vue'

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
   //console.log(data.urls.full)
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
      //console.log(data)
      crypto.value = data.name
      cryptoIcon.value = data.image.small
      cryptoPrice.value = data.market_data.current_price.aud
    })

// format currency
function currency(number: number) {
  return new Intl.NumberFormat('en-AU', { style: 'currency', currency: 'AUD', minimumFractionDigits: 0 }).format(number)
}


// clock
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

async function getIP() {
  console.log("using IP")  
  try {
        const response = await fetch('https://api.ipify.org?format=json');
        const data = await response.json();
        //console.log(data.ip);
        return data.ip
    } catch (error) {
        console.error('Error fetching IP address:', error);
    }
}

async function getGeolocation() {
  // get users IP to query their city
  const IPAddress = await getIP()
  // returns city, country, lat & long
  const url = `https://ip-geo-location10.p.rapidapi.com/ip?ip=${IPAddress}`;
  const options = {
    method: 'GET',
    headers: {
      'x-rapidapi-key': 'f9414091f9msh541ce9de998bbddp162ff3jsn0cbceac33b00',
      'x-rapidapi-host': 'ip-geo-location10.p.rapidapi.com'
    }
  };

  try {
    const response = await fetch(url, options);
    const data = await response.json();
    //console.log(data);
    return data.result
  } catch (error) {
    console.error(error);
  }
}

// prompt user to allow location
if (navigator.geolocation) {
  navigator.geolocation.getCurrentPosition(
      async ({coords})=> {
        console.log('coords: ', coords)
        getWeather(coords)
      }, 
      async err => {
        console.log(err)
        const coords = await getGeolocation()
        getWeather(coords)
      }, {
        enableHighAccuracy: true,
        timeout: 5000,
        maximumAge: 0,
      }
  )
}

// for location permission changes
navigator.permissions.query({name: 'geolocation'})
  .then(permission => {
      permission.onchange = async () => {
        // check for user permissions. If granted then get their currentPosition
        // otherwise we'll use their IP to get the city they're in
        //console.log("permission: ", permission.state)
        if (permission.state === 'granted') {
            navigator.geolocation.getCurrentPosition(
                async ({coords})=> {
                  getWeather(coords)
                }, 
                async err => {
                  console.log(err, "permissions not granted")
                  const coords = await getGeolocation()
                  getWeather(coords)
                }
            )
        } else {
          const coords = await getGeolocation()
          getWeather(coords)
        }
      }
  })

async function getWeather({latitude, longitude}: {latitude: number, longitude: number}) {
  //const url = `https://api.openweathermap.org/data/2.5/weather?lat=${latitude}&lon=${longitude}&units=metric&appid=${apiKey}`;
  // using openweather api through scrimba, no need for api key
  const url = `https://apis.scrimba.com/openweathermap/data/2.5/weather?lat=${latitude}&lon=${longitude}&units=metric`;
  const res = await fetch(url)
  const data = await res.json()
  //console.log('weather: ',data)
  weatherIcon.value = `https://openweathermap.org/img/wn/${data.weather[0].icon}.png`
  weather.value = `${data.name} ${Math.round(data.main.temp)}˚C`
  //return await res.json()
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
    <div v-if="weather" class="weather">
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

p {
  margin: 0;
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

</style>
