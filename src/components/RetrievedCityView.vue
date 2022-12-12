<template>
  <div class="flex flex-col flex-1 items-center bg-fixed bg-cover">
    <div
      class="p-6 bg-sky-600 bg-opacity-10 backdrop-blur-lg rounded drop-shadow-lg rounded-lg shadow-md dark:border-gray-700 mt-10">
      <div class="flex flex-col items-center text-white py-12">
        <h1 class="text-8xl mb-2">{{ route.params.city }}</h1>
        <p class="text-sm">
          {{
              new Date(weatherData?.currentTime).toLocaleDateString(
                "tr-tr",
                {
                  weekday: "short",
                  day: "2-digit",
                  month: "long",
                }
              )
          }}
          {{
              new Date(weatherData?.currentTime).toLocaleTimeString(
                "tr-tr",
                {
                  timeStyle: "short",
                }
              )
          }}
        </p>
        <div class="place-items-center text-6xl mb-8 text-rose-600" v-if="rising">&#x25B2;</div>
        <p class="text-8xl mb-8 place-items-center">
          {{ Math.round(weatherData?.current.temp) }}&deg
        </p>
        <div class="place-items-center text-6xl mb-8 text-sky-600" v-if="!rising">&#x25BC;</div>
        <p>
          Feels like
          {{ Math.round(weatherData?.current.feels_like) }} &deg;
        </p>
        <p class="capitalize">
          {{ weatherData?.current.weather[0].description }}
        </p>
        <img class="w-[150px] h-auto" :src="
          `http://openweathermap.org/img/wn/${weatherData?.current.weather[0].icon}@2x.png`
        " alt="" />
      </div>
    </div>
    <div class="max-w-screen-md w-full py-12">
      <div
        class=" p-6 bg-sky-600 bg-opacity-20 backdrop-blur-lg rounded drop-shadow-lg  rounded-lg shadow-md dark:border-gray-700">
        <div class="mx-8 text-white">
          <h2 class="mb-4">Hourly Weather</h2>
          <div class="flex gap-10 overflow-x-scroll">
            <div v-for="hourData in weatherData.hourly" :key="hourData.dt" class="flex flex-col gap-4 items-center">
              <p class="whitespace-nowrap text-md mx-2">
                {{
                    new Date(
                      hourData.currentTime
                    ).toLocaleTimeString("tr-tr", {
                      hour: "numeric",
                    })
                }}
              </p>
              <img class="w-auto h-[50px] object-cover" :src="
                `http://openweathermap.org/img/wn/${hourData.weather[0].icon}@2x.png`
              " alt="" />
              <p class="text-xl">
                {{ Math.round(hourData.temp) }}&deg;
              </p>
            </div>
          </div>
        </div>
      </div>
      <div class="max-w-screen-md w-full py-12">
        <div
          class=" p-6 bg-sky-600 bg-opacity-20 backdrop-blur-lg rounded drop-shadow-lg0 rounded-lg shadow-md dark:border-gray-700">
          <div class="mx-8 text-white">
            <h2 class="mb-4">7 Day Forecast</h2>
            <div v-for="day in weatherData?.daily" :key="day.dt" class="flex items-center">
              <p class="flex-1">
                {{
                    new Date(day.dt * 1000).toLocaleDateString(
                      "tr-tr",
                      {
                        weekday: "long",
                      }
                    )
                }}
              </p>
              <img class="w-[50px] h-[50px] object-cover" :src="
                `http://openweathermap.org/img/wn/${day.weather[0].icon}@2x.png`
              " alt="" />
              <div class="flex gap-2 flex-1 justify-end">
                <p>H: {{ Math.round(day.temp.max) }}</p>
                <p>L: {{ Math.round(day.temp.min) }}</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script setup>
import axios from "axios";
import allData from '../../requiredInfo.json'
import { onMounted, ref } from "vue";
import { useRoute, useRouter } from "vue-router";
const rising = ref(false)
const API_KEY = allData.CONFIGURATION_PARAMETERS[0].API_KEY
const route = useRoute();
onMounted(() => {
  //Change background according to weather.
  let background = document.getElementsByClassName('flex flex-col flex-1 items-center bg-fixed bg-cover')
  if (weatherData?.current.weather[0].description.includes("rain")) {
    background[0].classList.add("bg-[url('../img/SL_042221_42420_28.jpg')]")
  } else if (weatherData?.current.weather[0].description.includes("cloud")) {
    background[0].classList.add("bg-[url('../img/2548049.webp')]")
  } else if (weatherData?.current.weather[0].description.includes("snow") || weatherData?.current.weather[0].description.includes("drizzle")) {
    background[0].classList.add("bg-[url('../img/snowman-5k-snowfall-winter-wallpaper-preview.jpg')]")
  }
  else if (weatherData?.current.weather[0].description.includes("sky")) {
    background[0].classList.add("bg-[url('../img/clear-sky-hd-wallpaper.jpg')]")
  }
  else if (weatherData?.current.weather[0].description.includes("mist")) {
    background[0].classList.add("bg-[url('../img/pexels-lumn-167699.jpg')]")
  }
})
//Getting all the specific information about city.
const getWeatherData = async () => {
  try {
    const weatherData = await axios.get(
      `https://api.openweathermap.org/data/2.5/onecall?lat=${route.query.lat}&lon=${route.query.lng}&exclude={part}&appid=${API_KEY}&units=metric`
    );
    const localOffset = new Date().getTimezoneOffset() * 60000;
    const utc = weatherData.data.current.dt * 1000 + localOffset;
    weatherData.data.currentTime =
      utc + 1000 * weatherData.data.timezone_offset;
    weatherData.data.hourly.forEach((hour) => {
      const utc = hour.dt * 1000 + localOffset;
      hour.currentTime =
        utc + 1000 * weatherData.data.timezone_offset;
    });
    if (Math.round(weatherData.data.hourly[0].temp) < Math.round(weatherData.data.hourly[1].temp)) {
      rising.value = true
    } else if (Math.round(weatherData.data.hourly[0].temp) == Math.round(weatherData.data.hourly[1].temp)) {
      rising.value = null
    }
    return weatherData.data;
  } catch (err) {
    console.log(err);
  }
};
const weatherData = await getWeatherData();
const router = useRouter();
</script>
