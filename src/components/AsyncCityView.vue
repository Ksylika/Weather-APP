<template>
    <div class="flex flex-col flex-1 items-center">
        <div v-if="route.query.preview" class="text-white p-4 bg-weather-secondary w-full text-center">
            <p>
                Вы сейчас просматриваете этот город, нажмите на значок «+»
                чтобы начать отслеживать этот город
            </p>
        </div>
        <div class="flex flex-col items-center text-white py-12">
            <h1 class="text-4xl mb-2"> {{ route.params.city }} </h1>
            <p class="text-sm mb-12">
                {{
                    new Date(weatherData.current.time).toLocaleDateString(
                    "ru-RU",
                    {
                        weekday: "short",
                        day: "2-digit",
                        month: "long",
                    }
                    )
                }}
                {{
                    new Date(weatherData.current.time).toLocaleTimeString(
                    "ru-Ru",
                    {
                        timeStyle: "short",
                    }
                    )
                }}
            </p>
            <p class="text-8xl mb-8">
                {{ Math.round(weatherData.current.temperature_2m) }}&deg;
            </p>
            <div class="text-center">
                <p>
                    Ощущается как: {{ Math.round(weatherData.current.apparent_temperature)}}&deg;
                </p>
                <p class="capitalize"> {{ weatherCodes[weatherData.current.weather_code] }} </p>
                <i
                class='w-[150px] text-6xl h-auto wi '
                :class="weatherIcons[weatherData.current.weather_code]"
                ></i>
            </div>
            
        </div>
        
        <hr class="border-white border-opacity-10 border w-full">
        
        <div class="max-w-screen-md w-full py-12">
            <div class="mx-8 text-white">
                <h2 class="mb-4">Погода по часам</h2>
                
                    <div
                    ref="scrollContainer"  
                    class="flex gap-10 horizontal-scroll"
                    @wheel="handleScroll">
                    
                        <div  v-for="(hourData, index) in weatherData.hourly.time.slice(0, 24)" :key="index" class="flex flex-col gap-4 items-center">
                            <!-- {{ hourData }} -->
                            <p class="whitespace-nowrap text-md">
                                
                                {{
                                    new Date(
                                    hourData
                                    ).toLocaleTimeString("ru-Ru", {
                                        hour: "numeric",
                                    })
                                }}:00
                            </p>
                            <i
                            class="wi w-auto text-2xl h-[25px] object-cover"
                            :class="weatherIcons[weatherData.hourly.weather_code[index]]"
                            ></i>
                            
                            <p class="text-xl">
                                {{ Math.round(weatherData.hourly.temperature_2m[index]) }}&deg;
                            </p>
                        </div>
                    
                </div>
            </div>
        </div>
        
        <hr class="border-white border-opacity-10 border w-full">
        
        <div class="max-w-screen-md w-full py-12">
            <div class="mx-8 text-white">
                <h2 class="mb-4">Погода на 7 дней</h2>
                <div
                v-for="(daily, index) in weatherData.daily.time" :key="index"
                class="flex items-center"
                >
                <p class="flex-1">
                    {{
                        new Date(daily).toLocaleDateString(
                        "ru-Ru",
                        {
                            weekday: "long",
                        }
                        )
                    }}
                </p>
                <i
                class="wi w-auto text-4xl h-[50px] object-cover"
                :class="weatherIcons[weatherData.daily.weather_code[index]]"
                ></i>
                <div class="flex gap-2 flex-1 justify-end">
                    <p>Макс: {{ Math.round(weatherData.daily.temperature_2m_max[index]) }}</p>
                    <p>Мин: {{ Math.round(weatherData.daily.temperature_2m_min[index]) }}</p>
                </div>
            </div>
        </div>
    </div>
    <div class="flex items-center gap-2 py-12 text-white cursor-pointer duration-150 hover:text-red-500"
    @click="removeCity"
    >
    <i class="fa-solid fa-trash"></i>
    <p>Удалить город</p>
</div>
</div>
</template>

<script setup>
import axios from 'axios';
import { useRoute, useRouter } from 'vue-router';
import { ref } from 'vue';

const route = useRoute();
const getWeatherData = async () => {
    try {
        const weatherData = await axios.get(
        `https://api.open-meteo.com/v1/forecast?latitude=${route.query.lat}&longitude=${route.query.lon}&current=temperature_2m,apparent_temperature,weather_code&hourly=temperature_2m,apparent_temperature,weather_code&daily=weather_code,temperature_2m_max,temperature_2m_min,apparent_temperature_max,apparent_temperature_min&timezone=Europe/Moscow`
        );
        await new Promise((res) => setTimeout(res, 1000))
        return weatherData.data
    } catch (error) {
        console.log(error);
    }
}
const weatherData = await getWeatherData();
const weatherCodes = {
    0: "Ясно",
    1: "Частично облачно",
    2: "Частично облачно",
    3: "Частично облачно",
    45: "Туман",
    48: "Туман",
    51: "Морось",
    53: "Морось",
    55: "Морось",
    61: "Дождь",
    63: "Дождь",
    65: "Дождь",
    71: "Снег",
    73: "Снег",
    75: "Снег",
    95: "Гроза"
};
const weatherIcons = {
    0: "wi-day-sunny",          // Ясно
    1: "wi-day-cloudy",         // Частично облачно
    2: "wi-day-cloudy",         // Частично облачно
    3: "wi-cloudy",             // Облачно
    45: "wi-fog",               // Туман
    48: "wi-fog",               // Туман
    51: "wi-sprinkle",          // Морось
    53: "wi-sprinkle",          // Морось
    55: "wi-sprinkle",          // Морось
    61: "wi-rain",              // Дождь
    63: "wi-rain",              // Дождь
    65: "wi-rain",              // Дождь
    71: "wi-snow",              // Снег
    73: "wi-snow",              // Снег
    75: "wi-snow",              // Снег
    95: "wi-thunderstorm"       // Гроза
};
const router = useRouter();
const removeCity = () => {
    const cities = JSON.parse(localStorage.getItem('savedCities'));
    const updatedCities = cities.filter(city => city.id !== route.query.id);
    localStorage.setItem('savedCities', JSON.stringify(updatedCities));
    router.push({
        name: 'home',
    })
}

const scrollContainer = ref(null);

const handleScroll = (event) => {
  
  if (event.deltaY !== 0) {
    scrollContainer.value.scrollLeft += event.deltaY;
    event.preventDefault(); 
  }
};

</script>

<style scoped>
.horizontal-scroll {
    width: 100%; /* Ограничиваем ширину контейнера */
    overflow-x: scroll; /* Включаем горизонтальную прокрутку */
    white-space: nowrap; /* Не переносим содержимое на новую строку */
    scrollbar-width: none; /* Скрываем прокрутку для Firefox */
}

.horizontal-scroll::-webkit-scrollbar {
    display: none; /* Скрываем полосу прокрутки для Webkit (Chrome, Safari) */
}
</style>