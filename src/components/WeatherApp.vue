<script setup>
import { ref, onMounted, watch } from 'vue'
import axios from 'axios'
import AppHeader from './AppHeader.vue'
import AppFooter from './AppFooter.vue'
import LocationSelector from './LocationSelector.vue'
import WeatherDisplay from './WeatherDisplay.vue'

const regions = ref([])
const provinces = ref([])
const cities = ref([])

const selectedRegionCode = ref(null)
const selectedProvinceCode = ref(null)
const selectedCity = ref(null)

const weatherData = ref(null)
const loading = ref(false)
const error = ref(null)

// Replace with your API key
const apiKey = import.meta.env.VITE_WEATHER_API_KEY
const NCR_CODE = '130000000'
const METRO_MANILA_PSEUDO_CODE = 'MM000000' // Using a pseudo-code for Metro Manila

// Function to map conditions to Makin-Things weather-icons
const getWeatherIcon = (conditionText, isDay) => {
    const condition = conditionText.toLowerCase()
    const timeOfDay = isDay ? 'day' : 'night'

    // Mapping based on condition texts to Makin-Things icon names
    const iconMap = {
        'sunny': `clear-${timeOfDay}`,
        'clear': `clear-${timeOfDay}`,
        'partly cloudy': `cloudy-1-${timeOfDay}`,
        'cloudy': `cloudy`,
        'overcast': `cloudy-3-${timeOfDay}`,
        'mist': `fog`,
        'patchy rain possible': `rainy-1-${timeOfDay}`,
        'patchy snow possible': `snowy-1-${timeOfDay}`,
        'patchy sleet possible': `rain-and-sleet-mix`,
        'patchy freezing drizzle possible': `rain-and-sleet-mix`,
        'thundery outbreaks possible': `scattered-thunderstorms-${timeOfDay}`,
        'blowing snow': `snowy-3`,
        'blizzard': `snowy-3`,
        'fog': `fog`,
        'freezing fog': `frost`,
        'patchy light drizzle': `rainy-1`,
        'light drizzle': `rainy-1`,
        'freezing drizzle': `rain-and-sleet-mix`,
        'heavy freezing drizzle': `rain-and-sleet-mix`,
        'patchy light rain': `rainy-1-${timeOfDay}`,
        'light rain': `rainy-1-${timeOfDay}`,
        'moderate rain at times': `rainy-2-${timeOfDay}`,
        'moderate rain': `rainy-2-${timeOfDay}`,
        'heavy rain at times': `rainy-3-${timeOfDay}`,
        'heavy rain': `rainy-3-${timeOfDay}`,
        'light freezing rain': `rain-and-sleet-mix`,
        'moderate or heavy freezing rain': `rain-and-sleet-mix`,
        'light sleet': `rain-and-sleet-mix`,
        'moderate or heavy sleet': `rain-and-sleet-mix`,
        'patchy light snow': `snowy-1-${timeOfDay}`,
        'light snow': `snowy-1-${timeOfDay}`,
        'patchy moderate snow': `snowy-2-${timeOfDay}`,
        'moderate snow': `snowy-2-${timeOfDay}`,
        'patchy heavy snow': `snowy-3-${timeOfDay}`,
        'heavy snow': `snowy-3-${timeOfDay}`,
        'ice pellets': `hail`,
        'light rain shower': `rainy-1-${timeOfDay}`,
        'moderate or heavy rain shower': `rainy-2-${timeOfDay}`,
        'torrential rain shower': `rainy-3-${timeOfDay}`,
        'light sleet showers': `rain-and-sleet-mix`,
        'moderate or heavy sleet showers': `rain-and-sleet-mix`,
        'light snow showers': `snowy-1-${timeOfDay}`,
        'moderate or heavy snow showers': `snowy-2-${timeOfDay}`,
        'light showers of ice pellets': `hail`,
        'moderate or heavy showers of ice pellets': `hail`,
        'patchy light rain with thunder': `scattered-thunderstorms-${timeOfDay}`,
        'moderate or heavy rain with thunder': `thunderstorms`,
        'patchy light snow with thunder': `snowy-2-${timeOfDay}`,
        'moderate or heavy snow with thunder': `snowy-3-${timeOfDay}`,
    }

    const iconName = iconMap[condition] || 'cloudy' // Fallback to 'cloudy' if no match
    return `https://raw.githubusercontent.com/Makin-Things/weather-icons/master/animated/${iconName}.svg`
}

const getWeatherAdvice = (conditionText) => {
    const condition = conditionText.toLowerCase()
    const advice = {
        dos: [],
        donts: [],
    }

    if (condition.includes('sunny') || condition.includes('clear')) {
        advice.dos.push("Magsuot ng sunscreen para protektahan ang balat.")
        advice.dos.push("Uminom ng maraming tubig para manatiling hydrated.")
        advice.dos.push("Magsuot ng salamin sa araw at sumbrero.")
        advice.donts.push("Huwag tumitig nang direkta sa araw.")
        advice.donts.push("Iwasan ang matagal na pagbibilad sa araw, lalo na sa tanghali.")
    } else if (condition.includes('rain') || condition.includes('drizzle') || condition.includes('shower')) {
        advice.dos.push("Magdala ng payong o magsuot ng kapote.")
        advice.dos.push("Mag-ingat sa pagmamaneho dahil madulas ang kalsada.")
        advice.dos.push("Manatili sa loob ng bahay kung malakas ang ulan.")
        advice.donts.push("Huwag subukang tumawid sa baha.")
        advice.donts.push("Iwasan ang pagsilong sa ilalim ng puno kung may kidlat at kulog.")
    } else if (condition.includes('cloudy') || condition.includes('overcast')) {
        advice.dos.push("Magandang panahon para sa outdoor activities na hindi nangangailangan ng matinding sikat ng araw.")
        advice.dos.push("Magdala pa rin ng payong kung may tsansang umulan.")
        advice.donts.push("Huwag kalimutang mag-check ng weather update kung may balak lumabas.")
    } else if (condition.includes('thunder') || condition.includes('thundery')) {
        advice.dos.push("Manatili sa loob ng bahay o gusali.")
        advice.dos.push("I-unplug ang mga appliances para maiwasan ang pinsala mula sa power surge.")
        advice.dos.push("Makinig sa mga ulat panahon para sa mga babala.")
        advice.donts.push("Huwag gumamit ng landline phone.")
        advice.donts.push("Huwag maligo o gumamit ng tubig habang may kidlat.")
        advice.donts.push("Huwag sumilong sa ilalim ng puno.")
    } else if (condition.includes('fog') || condition.includes('mist')) {
        advice.dos.push("Mag-ingat sa pagmamaneho dahil sa limitadong visibility.")
        advice.dos.push("Gamitin ang iyong fog lights kung kinakailangan.")
        advice.donts.push("Huwag mag-overtake kung hindi sigurado sa daan.")
    } else {
        advice.dos.push("Laging i-check ang pinakabagong weather forecast.")
        advice.dos.push("Maging handa sa anumang pagbabago ng panahon.")
        advice.donts.push("Huwag balewalain ang mga weather advisory.")
    }

    return advice
}

const fetchRegions = async () => {
    try {
        const response = await axios.get('https://psgc.gitlab.io/api/regions/')
        regions.value = response.data
    } catch (err) {
        error.value = 'Error: Hindi ma-load ang mga rehiyon.'
        console.error(err)
    }
}

const fetchProvinces = async (regionCode) => {
    if (!regionCode) return
    try {
        const response = await axios.get(`https://psgc.gitlab.io/api/regions/${regionCode}/provinces/`)
        provinces.value = response.data
    } catch (err) {
        error.value = 'Error: Hindi ma-load ang mga probinsya.'
        console.error(err)
    }
}

const fetchCities = async (provinceCode) => {
    if (!provinceCode) return
    try {
        const response = await axios.get(`https://psgc.gitlab.io/api/provinces/${provinceCode}/cities-municipalities/`)
        cities.value = response.data
    } catch (err) {
        error.value = 'Error: Hindi ma-load ang mga lungsod/bayan.'
        console.error(err)
    }
}

const fetchNcrCities = async () => {
    try {
        const response = await axios.get(`https://psgc.gitlab.io/api/regions/${NCR_CODE}/cities-municipalities/`)
        cities.value = response.data
    } catch (err) {
        error.value = 'Error: Hindi ma-load ang mga lungsod para sa NCR.'
        console.error(err)
    }
}

const fetchWeather = async () => {
    if (!selectedCity.value) return
    loading.value = true
    error.value = null
    try {
        const weatherResponse = await axios.get(
            `https://api.weatherapi.com/v1/forecast.json?key=${apiKey}&q=${selectedCity.value.name}&days=3&aqi=no&alerts=yes`
        )
        weatherData.value = weatherResponse.data
    } catch (err) {
        error.value = 'Error: Hindi makuha ang weather data. Please try again later.'
        console.error(err)
    } finally {
        loading.value = false
    }
}

watch(selectedRegionCode, (newRegionCode) => {
    provinces.value = []
    cities.value = []
    selectedProvinceCode.value = null
    selectedCity.value = null
    weatherData.value = null

    if (newRegionCode) {
        if (newRegionCode === NCR_CODE) {
            provinces.value = [{ name: 'Metro Manila', code: METRO_MANILA_PSEUDO_CODE }]
            selectedProvinceCode.value = METRO_MANILA_PSEUDO_CODE
            fetchNcrCities()
        } else {
            fetchProvinces(newRegionCode)
        }
    }
})

watch(selectedProvinceCode, (newProvinceCode) => {
    cities.value = []
    selectedCity.value = null
    weatherData.value = null

    if (newProvinceCode && newProvinceCode !== METRO_MANILA_PSEUDO_CODE) {
        fetchCities(newProvinceCode)
    }
})

watch(selectedCity, (newCity) => {
    if (newCity) {
        fetchWeather()
    }
})

watch(cities, (newCities) => {
    if (selectedRegionCode.value === NCR_CODE && newCities.length > 0 && !selectedCity.value) {
        const manila = newCities.find(city => city.name === 'City of Manila')
        if (manila) {
            selectedCity.value = manila
        }
    }
})

const formatDate = (dateString) => {
    return new Date(dateString).toLocaleDateString('en-US', {
        weekday: 'short',
        month: 'short',
        day: 'numeric',
    })
}

const formatDateTime = (dateTimeString) => {
    if (!dateTimeString) return 'N/A';
    return new Date(dateTimeString).toLocaleString('en-US', {
        year: 'numeric',
        month: 'short',
        day: 'numeric',
        hour: '2-digit',
        minute: '2-digit',
        hour12: true,
    })
}

const formatTime = (dateTimeString) => {
    if (!dateTimeString) return 'N/A';
    return new Date(dateTimeString).toLocaleTimeString('en-US', {
        hour: '2-digit',
        minute: '2-digit',
        hour12: true,
    })
}

onMounted(async () => {
    await fetchRegions()
    selectedRegionCode.value = NCR_CODE
})
</script>

<template>
    <div class="min-h-screen bg-gradient-to-br from-blue-50 via-white to-green-50 font-sans">
        <main class="flex-grow p-4 max-w-6xl mx-auto w-full">
            <AppHeader :weatherData="weatherData" :getWeatherIcon="getWeatherIcon" />
            <LocationSelector :regions="regions" :provinces="provinces" :cities="cities"
                :selectedRegionCode="selectedRegionCode" :selectedProvinceCode="selectedProvinceCode"
                :selectedCity="selectedCity" @update:selectedRegionCode="selectedRegionCode = $event"
                @update:selectedProvinceCode="selectedProvinceCode = $event"
                @update:selectedCity="selectedCity = $event" />

            <WeatherDisplay :weatherData="weatherData" :loading="loading" :error="error" :selectedCity="selectedCity"
                :selectedRegionCode="selectedRegionCode" :getWeatherIcon="getWeatherIcon" :formatDate="formatDate"
                :formatDateTime="formatDateTime" :getWeatherAdvice="getWeatherAdvice" :formatTime="formatTime" />
        </main>

        <AppFooter />
    </div>
</template>

<style scoped>
@keyframes spin-slow {
    from {
        transform: rotate(0deg);
    }

    to {
        transform: rotate(360deg);
    }
}

.animate-spin-slow {
    animation: spin-slow 3s linear infinite;
}
</style>