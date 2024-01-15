<template>
  <div class="flex flex-col flex-1 items-center">
    <!-- Banner -->
    <div v-if="route.query.preview" class="bg-weather-secondary w-full text-center">
      <p class="text-sm text-white py-2 opacity-80 mx-8">
        You are currently previewing this city, click the <span class="text-sm mx-1"> + </span>
        icon to start tracking this city.
      </p>
    </div>
    <!-- Waether OverView -->
    <div class="flex flex-col justify items-center text-white py-12">
      <h1 class="text-6xl mb-2">{{ route.params.city }}</h1>
      <p class="text-sm">
        {{
          new Date(weatherData.currentTime).toLocaleDateString(
            "en-us",
            {
              weekday: "short",
              month: "long",
              day: "2-digit",
            }
          )
        }}
        {{
          new Date(weatherData.currentTime).toLocaleTimeString(
            "en-us",
            {
              timeStyle: "short",
            }
          )
        }}
      </p>
      <p class="text-8xl my-12">
        {{ Math.round(celsiusFunction(weatherData.current.temp)) }}&deg;
      </p>
      <p class="text-lg">
        Feels like
        {{ Math.round(celsiusFunction(weatherData.current.feels_like)) }} &deg;
      </p>
      <p class="capitalize">
        {{ weatherData.current.weather[0].description }}
      </p>
      <img class="w-[96px] h-auto mt-4" :src="`http://openweathermap.org/img/wn/${weatherData.current.weather[0].icon}@2x.png`
        " alt="weather status icon" />
    </div>
    <hr class="border-white border-opacity-20 border w-full">
    <!-- Hourly weather -->
    <div class="max-w-screen-md w-full py-20">
      <div class="mx-8 text-white">
        <h2 class="text-md mb-8">Hourly Weather</h2>
        <div class="flex gap-10 overflow-x-scroll">
          <div v-for="hourData in weatherData.hourly" :key="hourData.dt"
            class="flex flex-col item-center gap-2 p-8 mb-4 rounded-xl text-center">
            <p class="whitespace-nowrap text-sm">
              {{
                new Date(
                  hourData.currentTime
                ).toLocaleTimeString("en-us", {
                  hour: "numeric",
                })
              }}
            </p>
            <img class="w-auto h-[48px] object-cover" :src="`http://openweathermap.org/img/wn/${hourData.weather[0].icon}@2x.png`
                " alt="" />
            <p class="text-lg">
              {{ Math.round(celsiusFunction(hourData.temp)) }}&deg;
            </p>
          </div>
        </div>
      </div>
    </div>
    <hr class="border-white border-opacity-20 border w-full">
    <!-- Weekkly Weather -->
    <div class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="text-md mb-4">{{ `7 Day Forecast / ${route.params.city}` }}</h2>
        <div v-for="day in weatherData.daily" :key="day.dt" class="flex items-center">
          <p class="text-sm flex-1">
            {{
              new Date(day.dt * 1000).toLocaleDateString(
                "en-us",
                {
                  weekday: "long",
                }
              )
            }}
          </p>
          <img class="w-[48px] h-[48px] object-cover" :src="`http://openweathermap.org/img/wn/${day.weather[0].icon}@2x.png`
              " alt="weather status icon" />
          <div class="flex gap-2 flex-1 justify-end">
            <p class="text-sm">{{ `${Math.round(celsiusFunction(day.temp.max))} º` }} / {{
              `${Math.round(celsiusFunction(day.temp.min))} º`
            }}</p>
          </div>
        </div>
      </div>
    </div>
    <div v-if="isCityInLocalStorage" @click="removeCity"
      class="flex items-center gap-2 text-white cursor-pointer border-2 my-4 p-2 rounded-md hover:bg-weather-secondary duration-200">
      <p class="text-sm">Remove City</p>
    </div>
  </div>
</template>

<script setup>
import axios from "axios";
import { useRoute, useRouter } from "vue-router";
import { celsiusFunction } from "../helpers/celsiusFunction"
import { onMounted, ref } from "vue";
const route = useRoute();
const router = useRouter()

const getWeatherData = async () => {
  try {
    const weatherData = await axios.get(
      `https://api.openweathermap.org/data/2.5/onecall?lat=${route.query.lat}&lon=${route.query.lng}&exclude={part}&appid=7efa332cf48aeb9d2d391a51027f1a71&units=imperial`
    );

    // cal current date & time
    const localOffset = new Date().getTimezoneOffset() * 60000;
    const utc = weatherData.data.current.dt * 1000 + localOffset;
    weatherData.data.currentTime =
      utc + 1000 * weatherData.data.timezone_offset;

    // cal hourly weather offset
    weatherData.data.hourly.forEach((hour) => {
      const utc = hour.dt * 1000 + localOffset;
      hour.currentTime =
        utc + 1000 * weatherData.data.timezone_offset;
    });

    // Flicker Delay
    await new Promise((res) => setTimeout(res, 500))

    return weatherData.data;
  } catch (err) {
    console.log(err);
  }
};

const weatherData = await getWeatherData();

const removeCity = () => {
  const cities = JSON.parse(localStorage.getItem("savedCities"))
  const updateCities = cities.filter((city) => city.id !== route.query.id)
  localStorage.setItem("savedCities", JSON.stringify(updateCities))
  router.push({
    name: "home"
  })
}

const isCityInLocalStorage = ref(false)
onMounted(() => {
  const cities = JSON.parse(localStorage.getItem('savedCities')) || [];
  isCityInLocalStorage.value = cities.some((city) => city.id === route.query.id)
})

</script>