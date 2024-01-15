<template>
  <header class="sticky top-0 bg-weather-primary shadow-lg">
    <nav class="container flex flex-col sm:flex-row items-center gap-4 text-white py-6">
      <RouterLink :to="{ name: 'home' }">
        <div class="flex item-center gap-3">
          <img src="/icons/sun.svg" alt="Sun icon" width="32">
          <p class="text-2xl">Brasero´s Weather</p>
        </div>
      </RouterLink>

      <div class="flex gap-3 flex-1 justify-end">
        <img @click="toggleModal" src="/icons/info.svg" alt="Info icon" width="32" class="cursor-pointer">
        <img @click="addCity" v-if="route.query.preview" src="/icons/plus.svg" alt="Plus icon" width="32"
          class="cursor-pointer">
      </div>
    </nav>
  </header>
  <BaseModal :modalActive="modalActive" @close-modal="toggleModal">
    <div class="text-black">
      <h1 class="text-2xl mb-2 font-semibold">About :</h1>
      <p class="mb-4">
        The Local Weather allows you to track the current and
        future weather of cities of your choosing.
      </p>
      <h2 class="text-2xl mb-2 font-semibold">How it works :</h2>
      <ol class="list-decimal list-inside mb-4">
        <li>
          Search for your city by entering the name into the
          search bar.
        </li>
        <li>
          Select a city within the results, this will take
          you to the current weather for your selection.
        </li>
        <li>
          Track the city by clicking on the "+" icon in the
          top right. This will save the city to view at a
          later time on the home page.
        </li>
      </ol>

      <h2 class="text-2xl mb-2 font-semibold">Removing a city :</h2>
      <p>
        If you no longer wish to track a city, simply select
        the city within the home page. At the bottom of the
        page, there will be am option to delete the city.
      </p>
    </div>
  </BaseModal>
</template>

<script setup>
import { RouterLink, useRoute, useRouter } from "vue-router";
import { uid } from "uid";
import { ref } from "vue";
import BaseModal from "./BaseModal.vue";

const savedCities = ref([]);
const route = useRoute();
const router = useRouter();
const addCity = () => {
  if (localStorage.getItem("savedCities")) {
    savedCities.value = JSON.parse(
      localStorage.getItem("savedCities")
    );
  }

  const locationObj = {
    id: uid(),
    state: route.params.state,
    city: route.params.city,
    coords: {
      lat: route.query.lat,
      lng: route.query.lng,
    },
  };

  savedCities.value.push(locationObj);
  localStorage.setItem(
    "savedCities",
    JSON.stringify(savedCities.value)
  );

  let query = Object.assign({}, route.query);
  delete query.preview;
  query.id = locationObj.id
  router.replace({ query });
};

const modalActive = ref(null);
const toggleModal = () => {
  modalActive.value = !modalActive.value;
};
</script>

