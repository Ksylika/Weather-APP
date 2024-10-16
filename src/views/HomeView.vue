<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input type="text"
      v-model="searchQuery"
      @input="getSearchResults"
      placeholder="Поиск города или штата" 
      class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]">
      <ul class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
      v-if="searchResults"
      >
      <p v-if="searchErorr" class="text-red-500">Не удалось найти город или штат</p>
      <p v-if="!searchErorr && searchResults.length === 0">Ничего не найдено</p>
      <template v-else>
        <li v-for="searchResult in searchResults" :key="searchResult.id"
        class="py-2 cursor-pointer"
        @click="previewCity(searchResult)"
        >
        {{ searchResult.display_name }}
      </li>
    </template>
    
  </ul>
</div>
<div class="flex flex-col gap-4">
  <Suspense>
    <CityList/>
    <template #fallback>
      <CityCardSkeleton/>
    </template>
  </Suspense>
</div>
</main>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'
import { useRouter } from 'vue-router'
import CityList from '../components/CityList.vue'
import CityCardSkeleton from '../components/CityCardSkeleton.vue';

const router = useRouter()

const previewCity = (searchResult) => {
  const [city, state] = searchResult.display_name.split(", ")
  router.push({name: 'cityView', 
  params: { city: city, state: state }, 
  query: {
    lat: searchResult.lat,
    lon: searchResult.lon,
    preview: true,
  }
}
)
}


const searchQuery= ref("")
const queryTimeout = ref(null)
const searchResults = ref(null)
const searchErorr = ref(null)

const getSearchResults = () => {
  clearTimeout(queryTimeout.value)
  queryTimeout.value = setTimeout( async () => {
    if (searchQuery.value != "") {
      try {
        const result = await axios.get(`https://nominatim.openstreetmap.org/search?city=${searchQuery.value}&format=json`)
        searchResults.value = result.data;
        console.log(searchResults.value)
      } catch (error) {
        searchErorr.value = true;
      }
      return;
    };
    
    searchResults.value = null;
    
  }, 300)
}
</script>
