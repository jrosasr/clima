<script setup>
import { ref, onMounted, computed } from 'vue'

const lat = ref(null)
const lon = ref(null)

const blurLevel = computed(() => {
    if (info.value) {
        if (info.value.weather.main === "Clear") return "backdrop-blur-[5px]";
        if (info.value.weather.main === "Atmosphere") return "backdrop-blur-[10px]";
        if (info.value.weather.main === "Clouds") return "backdrop-blur-[15px]";
        if (info.value.weather.main === "Snow") return "backdrop-blur-[30px]";
        if (info.value.weather.main === "Drizzle") return "backdrop-blur-[50px]";
        if (info.value.weather.main === "Rain") return "backdrop-blur-[70px]";
        if (info.value.weather.main === "Thunderstorm") return "backdrop-blur-[80px]";
    }

    return "backdrop-blur-[5px]"

})

const animations = ref({
    Thunderstorm: '/clima/animated/thunder.svg',
    Drizzle: '/clima/animated/rainy-2.svg',
    Rain: '/clima/animated/rainy-7.svg',
    Snow: '/clima/animated/snowy-6.svg',
    Clear: '/clima/animated/day.svg',
    Atmosphere: '/clima/animated/weather.svg',
    Clouds: '/clima/animated/cloudy-day-1.svg',
})

const info = ref(null)

onMounted(() => {
    if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition((position) => {
            lat.value = position.coords.latitude
            lon.value = position.coords.longitude
            const key = "e09dca86339db8efde3edff44588b821"

            const res = fetch(
                `https://api.openweathermap.org/data/2.5/weather?lat=${lat.value}&lon=${lon.value}&appid=${key}&lang=es`
            )
                .then(res => res.json())
                .then(data => {
                    info.value = {
                        country: data.sys.country,
                        temp: (data.main.temp - 273.15).toFixed(),
                        weather: {
                            description: data.weather[0].description,
                            main: data.weather[0].main
                        },
                        location: data.name,
                        wind: {
                            gust: data.wind.gust,
                            deg: data.wind.deg,
                            speed: data.wind.speed
                        },
                        main: {
                            humidity: data.main.humidity
                        },
                        icon: animations.value[data.weather[0].main] ?? "/base/animated/cloudy-day-1.svg"
                    }
                })
                .catch(error => {
                    console.log(error)
                });
        });
    }
})

</script>

<template>
    <section id="widget-clima" class="flex justify-center rounded-lg">
        <div class="flex flex-wrap justify-center gap-10 items-center my-20 p-0 md:p-12 rounded-lg w-full md:w-8/12" :class="`${blurLevel} ${!info ? ' h-[19rem]' : 'h-auto'}`">
            <div v-if="info" class="items-center">
                <div class="flex justify-center">
                    <img :src="info.icon" alt="Icono del clima" class="w-60 h-60">
                </div>
                <div class="flex flex-col items-center text-center mb-8 text-slate-900">
                    <span class="text-4xl md:text-5xl uppercase">{{ info.weather.description }}</span>
                    <span class="text-4xl md:text-3xl">{{ info.temp }} °C</span>
                </div>
            </div>
            <div v-if="info" class="text-slate-900">
                <p class="text-4xl">{{ info.location }}, {{ info.country }}</p>
                <ul class="text-left my-8 text-xl">
                    <li class="text-2xl"><span class="text-xl">Humedad:</span> {{ info.main.humidity }} %</li>
                    <li class="text-2xl"><span class="text-xl">Vel. Viento:</span> {{ info.wind.speed }} Mk/h</li>
                    <li class="text-2xl"><span class="text-xl">Grados:</span> {{ info.wind.deg }}</li>
                </ul>
            </div>
        </div>
    </section>
</template>