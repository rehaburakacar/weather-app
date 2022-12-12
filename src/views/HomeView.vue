<template>
  <main class="container text-white background">
    <div class="flex p-4 mb-4 text-sm text-red-700 bg-red-100 rounded-lg dark:bg-red-200 dark:text-red-800 mt-3"
      role="alert" v-if="!addedSuccesfully">
      <svg aria-hidden="true" class="flex-shrink-0 inline w-5 h-5 mr-3" fill="currentColor" viewBox="0 0 20 20"
        xmlns="http://www.w3.org/2000/svg">
        <path fill-rule="evenodd"
          d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-7-4a1 1 0 11-2 0 1 1 0 012 0zM9 9a1 1 0 000 2v3a1 1 0 001 1h1a1 1 0 100-2v-3a1 1 0 00-1-1H9z"
          clip-rule="evenodd"></path>
      </svg>
      <span class="sr-only">Danger</span>
      <div>
        <span class="font-medium">City cannot be Added to favourites!:</span>
        <ul class="mt-1.5 ml-4 text-red-700 list-disc list-inside">
          <li>Same city cannot be added twice to favourite list.</li>
        </ul>
      </div>
    </div>
    <div class="flex p-4 mb-4 text-sm text-red-700 bg-red-100 rounded-lg dark:bg-red-200 dark:text-red-800 mt-3"
      role="alert" v-if="networkError">
      <svg aria-hidden="true" class="flex-shrink-0 inline w-5 h-5 mr-3" fill="currentColor" viewBox="0 0 20 20"
        xmlns="http://www.w3.org/2000/svg">
        <path fill-rule="evenodd"
          d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-7-4a1 1 0 11-2 0 1 1 0 012 0zM9 9a1 1 0 000 2v3a1 1 0 001 1h1a1 1 0 100-2v-3a1 1 0 00-1-1H9z"
          clip-rule="evenodd"></path>
      </svg>
      <span class="sr-only">Danger</span>
      <div>
        <span class="font-medium">There may be network issues!:</span>
        <ul class="mt-1.5 ml-4 text-red-700 list-disc list-inside">
          <li>Please check the internet connection and API Key.</li>
        </ul>
      </div>
    </div>
    <div>
      <button type="button"
        class="bg-sky-600 bg-opacity-20 backdrop-blur-lg mt-5 inline-flex w-full justify-center rounded-md px-4 py-2 text-sm font-medium text-whiteshadow-sm mt-3"
        id="menu-button" aria-expanded="true" aria-haspopup="true" @click="openFrequency()">
        Change Frequency
        <svg class="-mr-1 ml-2 h-5 w-5" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor"
          aria-hidden="true">
          <path fill-rule="evenodd"
            d="M5.23 7.21a.75.75 0 011.06.02L10 11.168l3.71-3.938a.75.75 0 111.08 1.04l-4.25 4.5a.75.75 0 01-1.08 0l-4.25-4.5a.75.75 0 01.02-1.06z"
            clip-rule="evenodd" />
        </svg>
      </button>
    </div>
    <div
      class="bg-sky-600 bg-opacity-10 backdrop-blur-lg z-10 mt-2 w-50 justify-center rounded-md shadow-lg ring-1 ring-black ring-opacity-5 focus:outline-none"
      role="menu" aria-orientation="vertical" aria-labelledby="menu-button" tabindex="-1">
      <div class="py-1" role="none" v-for="frequency in frequencies" :key="frequency"
        :style="[closeFrequency ? { 'display': 'block' } : { 'display': 'none' }]">
        <a href="#" class="text-white block px-4 py-2 text-white" role="menuitem" tabindex="-1" id="menu-item-0"
          @click="changeFrequency(frequency)">{{ frequency }}</a>
      </div>
    </div>
    <div class="pt-4 mb-8 relative">
      <div class="relative inline-block text-left">
      </div>
      <input type="text" v-model="searchQuery" @input="getSearchResults" placeholder="Search for a city or state"
        class="bg-sky-600 bg-opacity-20 backdrop-blur-lg rounded-lg py-2 px-2 text-white  w-full focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
        @blur="closeSuggest()" @focus="openSuggest()" style="z-index= -1" />
      <ul
        class="z-50 absolute rounded-lg bg-sky-600 bg-opacity-20 backdrop-blur-lg text-white w-full shadow-md py-2 px-1 top[15px]"
        id="search" v-if="mapboxSearchResults">
        <p class="py-2" v-if="searchError">
          Sorry, something went wrong, please try again.
        </p>
        <p class="py-2" v-if="!searchError && mapboxSearchResults.length === 0">
          No results match your query, try a different term.
        </p>
        <template v-else>
          <li v-for="searchResult in mapboxSearchResults" :key="searchResult.id" class="py-2 cursor-pointer"
            @click="previewCity(searchResult, false)">
            {{ searchResult.place_name }}
          </li>
        </template>
      </ul>
      <p v-if="cities.length === 0" class="mt-2">
        Welcome to Vue Weather App. To add locations to favourite, please search a city.
      </p>
    </div>
    <div class="flex flex-col gap-4 mb-5">
      <div
        class="flex py-6 px-3 bg-sky-600 bg-opacity-20 backdrop-blur-lg rounded drop-shadow-lg rounded-md shadow-md cursor-pointer"
        v-for="cities in cities" :key="cities">
        <div class="flex flex-col flex-1" @click="goToCity(cities)">
          <h2 class="text-3xl">{{ cities?.name[0] }}</h2>
          <h3>{{ cities?.name[1] }}</h3>
        </div>

        <div class="flex flex-col gap-2 rounded-lg">
          <button @click="deleteCity(cities)" v-if="cities?.from != 'JSON'">
            Delete City
          </button>
          <p class="text-3xl self-end">
            {{ Math.round(cities.weather?.main.temp) }}&deg;
          </p>
          <div class="flex gap-2">
            <span class="text-xs">
              H:
              {{ Math.round(cities?.weather?.main.temp_max) }}
            </span>
            <span class="text-xs">
              L:
              {{ Math.round(cities.weather?.main.temp_min) }}
            </span>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>
<script setup>
import { onMounted, ref, watch } from "vue";
import axios from "axios";
import allData from '../../requiredInfo.json'
import { useRouter } from "vue-router";
const closeFrequency = ref(false)
var cities = ref([])
const API_KEY = allData.CONFIGURATION_PARAMETERS[0].API_KEY
const MAPBOX_API_KEY = allData.CONFIGURATION_PARAMETERS[1].MAPBOX_API_KEY
var addedSuccesfully = ref(true)
const frequencies = ref([])
const networkError = ref(false)
const changeFreq = ref(200000)
frequencies.value.push('Every Minute')
frequencies.value.push('Every Hour')
const router = useRouter();
onMounted(async () => {
  let background = document.getElementsByClassName('flex flex-col min-h-screen bg-fixed bg-cover')
  background[0].classList.add("bg-[url('../img/clear-air-fly-fantastic-travel.jpg')]")
  //Getting cities 
  if (localStorage.getItem('cities') !== null) {
    cities.value = JSON.parse(localStorage.getItem('cities'))
  }
  //Getting cities from JSON configuration file.
  allData.cities.forEach((city) => {
    previewCity(city, true)
  });
  setTemperature()
})
function changeFrequency(frequency) {
  switch (frequency) {
    case ('Every Minute'):
      changeFreq.value = 60000
      break;
    case ('Every Hour'):
      changeFreq.value = 3600000
      break;
  }
  setTimeout(() => {
    closeFrequency.value = false
  }, 75);
  setInterval(async () => {
    for (const element of cities.value) {
      await setTemperature(element, false)
    }
  }, changeFreq.value);
}
function openFrequency() {
  closeFrequency.value = true
}
const previewCity = async (searchResult, fromJson) => {
  let temp
  if (!fromJson) {
    var jsonObject = {
      id: searchResult.id,
      name: searchResult.place_name.split(","),
      lat: searchResult.geometry.coordinates[1],
      lng: searchResult.geometry.coordinates[0],
      from: "Storage"
    }
  } else {
    var jsonObject = {
      id: searchResult.id,
      name: searchResult.name.split(","),
      lat: searchResult.lat,
      lng: searchResult.lng,
      from: "JSON"
    }
  }
  //Check if city is present in pre-added cities or not.
  const found = cities.value.find(element => element.name[0] == jsonObject.name[0]);
  if (found == undefined) {
    cities.value.push(jsonObject);
    addedSuccesfully.value = true
    setTemperature()
  } else if (fromJson && found != undefined) {
    addedSuccesfully.value = true
  } else {
    addedSuccesfully.value = false
  }
  if (searchResult.place_name !== undefined) {
    const [city, state] = searchResult.place_name.split(",");
    localStorage.setItem('cities', JSON.stringify(cities.value))
  }
};
function closeSuggest() {
  var search = document.getElementById('search')
  setTimeout(() => {
    search.style.display = 'none'
  }, 125)
}
//Helps to change temperature dynamically
async function setTemperature() {
  cities.value.forEach((element) => {
    element.weather = null
  })
  const requests = [];
  cities.value.forEach((city) => {
    requests.push(
      axios.get(
        `https://api.openweathermap.org/data/2.5/weather?lat=${city.lat}&lon=${city.lng}&appid=${API_KEY}&units=metric`
      )
    );
  });
  const weatherData = await Promise.all(requests);
  weatherData.forEach((value, index) => {
    cities.value[index].weather = value.data;
  })
}
function deleteCity(city) {
  cities.value = cities.value.filter(res => res.id !== city.id)
  localStorage.setItem('cities', JSON.stringify(cities.value))
}
function openSuggest() {
  var search = document.getElementById('search')
  search.style.display = 'block'
}
//When user wants to retrieve any specific city information, app will direct to url.
async function goToCity(x) {
  const weatherData = await axios.get(
    `https://api.openweathermap.org/data/2.5/onecall?lat=${x.lat}&lon=${x.lng}&exclude={part}&appid=${API_KEY}&units=metric`
  );
  router.push({
    name: "cityView",
    params: { state: x.name[1], city: x.name[0] },
    query: {
      lat: x.lat,
      lng: x.lng,
      preview: true,
    },
  });
}
const searchQuery = ref("");
const queryTimeout = ref(null);
const mapboxSearchResults = ref(null);
const searchError = ref(null);
//Help to get city search results from Mapbox API.
const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== "") {
      try {
        const result = await axios.get(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${MAPBOX_API_KEY}&types=place`
        );
        mapboxSearchResults.value = result.data.features;
      } catch {
        searchError.value = true;
      }
      return;
    }
    mapboxSearchResults.value = null;
  }, 300);
};
</script>
<style scoped>
::placeholder {
  /* Chrome, Firefox, Opera, Safari 10.1+ */
  color: white;
  opacity: 1;
  /* Firefox */
}
</style>
