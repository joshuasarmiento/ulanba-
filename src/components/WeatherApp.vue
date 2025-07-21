<script setup>
import { ref, onMounted, watch } from 'vue'
import axios from 'axios'

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

onMounted(async () => {
    await fetchRegions()
    selectedRegionCode.value = NCR_CODE
})
</script>

<template>
    <div class="min-h-screen bg-gray-100 font-sans">
        <!-- Header -->
        <header class="bg-gray-800 text-white p-4 shadow-sm flex items-center justify-center">
            <img src="../assets/haze_animated.svg"
                class="h-12 w-12" alt="Logo" />
            <h1 class="text-2xl md:text-3xl font-bold">UlanBa?</h1>
        </header>

        <!-- Main Content -->
        <main class="flex-grow p-4 max-w-6xl mx-auto w-full">
            <!-- Location Selector -->
            <section class="mb-6">
                <h2 class="text-xl font-bold mb-3 text-gray-800">Saan ka?</h2>
                <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                    <select v-model="selectedRegionCode"
                        class="p-3 border-2 border-gray-300 rounded-lg shadow-sm focus:border-blue-500 focus:ring focus:ring-blue-200 transition">
                        <option :value="null" disabled>Rehiyon</option>
                        <option v-for="region in regions" :key="region.code" :value="region.code">{{ region.name }}
                        </option>
                    </select>
                    <select v-model="selectedProvinceCode" :disabled="!selectedRegionCode"
                        class="p-3 border-2 border-gray-300 rounded-lg shadow-sm focus:border-blue-500 focus:ring focus:ring-blue-200 transition">
                        <option :value="null" disabled>Probinsya</option>
                        <option v-for="province in provinces" :key="province.code" :value="province.code">{{
                            province.name }}</option>
                    </select>
                    <select v-model="selectedCity" :disabled="!selectedProvinceCode"
                        class="p-3 border-2 border-gray-300 rounded-lg shadow-sm focus:border-blue-500 focus:ring focus:ring-blue-200 transition">
                        <option :value="null" disabled>Lungsod / Bayan</option>
                        <option v-for="city in cities" :key="city.code" :value="city">{{ city.name }}</option>
                    </select>
                </div>
            </section>

            <!-- Loading / Error -->
            <div v-if="loading" class="text-center h-[60vh] p-10 flex flex-col items-center justify-center">
                <img src="https://raw.githubusercontent.com/Makin-Things/weather-icons/master/animated/dust.svg"
                    alt="Searching Icon" class="w-24 h-auto mb-4" />
                <p class="text-gray-600 text-lg">Loading weather data...</p>
            </div>
            <div v-if="error" class="text-center h-[60vh] p-10 flex flex-col items-center justify-center">
                <img src="https://raw.githubusercontent.com/Makin-Things/weather-icons/master/animated/severe-thunderstorm.svg"
                    alt="Error Icon" class="w-24 h-auto mb-4" />
                <p class="text-red-500 bg-red-100 p-4 rounded-lg">{{ error }}</p>
            </div>

            <!-- Searching for City Selection -->
            <div v-if="!weatherData && !loading && !error && selectedRegionCode && !selectedCity"
                class="text-center h-[60vh] p-10 flex flex-col items-center justify-center">
                <img src="https://raw.githubusercontent.com/Makin-Things/weather-icons/master/animated/dust.svg"
                    alt="Searching Icon" class="w-24 h-auto mb-4" />
                <p class="text-gray-600 text-lg">Searching for cities... Please select a Lungsod / Bayan.</p>
            </div>

            <!-- Main Weather Data -->
            <div v-if="weatherData && !loading">
                <!-- Weather Alerts -->
                <section v-if="weatherData.alerts && weatherData.alerts.alert.length" class="mb-6">
                    <h2 class="text-xl font-bold mb-3 text-red-600">Weather Alerts</h2>
                    <div v-for="(alert, index) in weatherData.alerts.alert" :key="index"
                        class="bg-red-100 border-l-4 border-red-500 text-red-700 p-4 rounded-lg shadow-sm mb-4 space-y-2">
                        <p class="font-bold text-lg">{{ alert.headline }}</p>
                        <p><span class="font-semibold">Event:</span> {{ alert.event }}</p>
                        <p><span class="font-semibold">Description:</span> {{ alert.desc }}</p>
                        <p v-if="alert.instruction"><span class="font-semibold">Instruction:</span> {{ alert.instruction
                            }}</p>
                        <div class="grid grid-cols-2 md:grid-cols-4 gap-2 text-sm mt-2">
                            <p><span class="font-semibold">Severity:</span> {{ alert.severity }}</p>
                            <p><span class="font-semibold">Urgency:</span> {{ alert.urgency }}</p>
                            <p><span class="font-semibold">Certainty:</span> {{ alert.certainty }}</p>
                            <p><span class="font-semibold">Category:</span> {{ alert.category }}</p>
                        </div>
                        <div class="grid grid-cols-1 md:grid-cols-2 gap-2 text-sm mt-2">
                            <p><span class="font-semibold">Effective:</span> {{ formatDateTime(alert.effective) }}</p>
                            <p><span class="font-semibold">Expires:</span> {{ formatDateTime(alert.expires) }}</p>
                        </div>
                    </div>
                </section>

                <!-- LPA & Typhoon Advisory -->
                <section class="mb-6">
                    <h2 class="text-xl font-bold mb-3 text-amber-800">LPA & Typhoon Advisory</h2>
                    <div class="bg-orange-100 border-l-4 border-orange-500 text-orange-700 p-4" role="alert">
                        <div class="font-bold mb-2">Para sa pinakahuling update tungkol sa Low Pressure Area (LPA) o
                            Bagyo, laging sumangguni sa opisyal na anunsyo ng PAGASA.</div>
                        <div class="flex flex-col gap-1">
                            <a href="https://www.pagasa.dost.gov.ph/weather/weather-advisory"
                            class="text-blue-600 hover:underline font-semibold" target="_blank"
                            rel="noopener noreferrer">
                            Tingnan ang Latest PAGASA Weather Advisory
                        </a>
                         <a href="https://www.pagasa.dost.gov.ph/flood"
                            class="text-blue-600 hover:underline font-semibold" target="_blank"
                            rel="noopener noreferrer">
                            PAGASA Flood Information
                        </a>
                        </div>
                        <div class="mt-4">
                            <h3 class="font-bold mb-1">Mga Dapat Tandaan:</h3>
                            <ul class="list-disc list-inside text-sm">
                                <li>Manatiling updated sa ulat-panahon mula sa PAGASA.</li>
                                <li>Ihanda ang inyong emergency go-bag.</li>
                                <li>Iwasan ang mga lugar na peligroso sa baha at landslide.</li>
                                <li>Maging alerto sa mga posibleng pagbabago ng panahon.</li>
                            </ul>
                        </div>
                    </div>
                </section>

                <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                    <!-- Current Weather -->
                    <div class="lg:col-span-1 bg-white p-6 rounded-xl shadow-sm flex flex-col items-center text-center">
                        <h2 class="text-xl font-bold text-gray-800 mb-4">Panahon Ngayon sa {{ selectedCity.name }}</h2>
                        <p class="text-5xl font-extrabold text-gray-900">{{ weatherData.current.temp_c }}°C</p>
                        <p class="text-gray-600 mb-2">Feels like: {{ weatherData.current.feelslike_c }}°C</p>
                        <div class="flex items-center">
                            <img :src="getWeatherIcon(weatherData.current.condition.text, weatherData.current.is_day)"
                                alt="Weather Icon" class="w-32 h-auto" />
                            <p class="text-lg capitalize font-semibold text-gray-700">{{
                                weatherData.current.condition.text }}</p>
                        </div>
                        <div class="flex justify-around w-full mt-4 text-sm text-gray-600">
                            <div class="flex items-center">
                                <svg class="w-6 h-6 mr-2 text-blue-500" fill="none" stroke="currentColor"
                                    viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                        d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"></path>
                                </svg>
                                <span>{{ weatherData.current.humidity }}% Humidity</span>
                            </div>
                            <div class="flex items-center">
                                <svg class="w-6 h-6 mr-2 text-blue-500" fill="none" stroke="currentColor"
                                    viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                        d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z">
                                    </path>
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                        d="M15 11a3 3 0 11-6 0 3 3 0 016 0z"></path>
                                </svg>
                                <span>{{ weatherData.current.wind_kph }} kph Wind</span>
                            </div>
                        </div>
                    </div>

                    <!-- 3-Day Forecast -->
                    <div class="lg:col-span-2 bg-white p-6 rounded-xl shadow-sm">
                        <h2 class="text-xl font-bold text-gray-800 mb-4">3-Day Forecast</h2>
                        <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                            <div v-for="day in weatherData.forecast.forecastday" :key="day.date"
                                class="text-center p-4 bg-gray-50 rounded-lg">
                                <h3 class="font-bold text-gray-800">{{ formatDate(day.date) }}</h3>
                                <img :src="getWeatherIcon(day.day.condition.text, true)" alt="Weather Icon"
                                    class="w-16 my-2 h-auto mx-auto" />
                                <p class="font-semibold text-lg">{{ day.day.maxtemp_c }}° / {{ day.day.mintemp_c }}°</p>
                                <p class="text-sm text-gray-600 capitalize">{{ day.day.condition.text }}</p>
                                <div class="flex items-center justify-center mt-2 text-sm text-blue-600">
                                    <svg class="w-4 h-4 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24"
                                        xmlns="http://www.w3.org/2000/svg">
                                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                            d="M5.636 18.364a9 9 0 010-12.728m12.728 0a9 9 0 010 12.728m-9.9-1.414v.001M9 12h.01M15 12h.01M12 9v.01M8.222 16.778L12 13.001l3.778 3.777M12 3v.01">
                                        </path>
                                    </svg>
                                    <span>{{ day.day.daily_chance_of_rain }}% Chance of Rain</span>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- PAGASA Synopsis -->
                <section class="mt-6">
                    <div class="bg-white p-4 rounded-lg shadow-sm text-sm text-gray-600">
                        <h2 class="font-bold text-gray-800 mb-2">Weather Synopsis (PAGASA)</h2>
                        <p>
                            Para sa opisyal na ulat panahon mula sa PAGASA, laging bisitahin ang kanilang website:
                            <a href="https://www.pagasa.dost.gov.ph/weather" class="text-blue-600 hover:underline"
                                target="_blank" rel="noopener noreferrer">PAGASA Weather Forecast</a>.
                        </p>
                    </div>
                </section>

                <!-- Weather Advice -->
                <section v-if="weatherData" class="mt-6">
                    <div class="bg-white p-6 rounded-xl shadow-sm">
                        <h2 class="text-xl font-bold text-gray-800 mb-4">Payo sa Panahon</h2>
                        <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                            <div>
                                <h3 class="font-bold text-lg text-green-600 mb-2">Mga Dapat Gawin (Do's)</h3>
                                <ul class="list-disc list-inside space-y-1 text-gray-700">
                                    <li v-for="(item, index) in getWeatherAdvice(weatherData.current.condition.text).dos"
                                        :key="index">{{ item }}</li>
                                </ul>
                            </div>
                            <div>
                                <h3 class="font-bold text-lg text-red-600 mb-2">Mga Hindi Dapat Gawin (Don'ts)</h3>
                                <ul class="list-disc list-inside space-y-1 text-gray-700">
                                    <li v-for="(item, index) in getWeatherAdvice(weatherData.current.condition.text).donts"
                                        :key="index">{{ item }}</li>
                                </ul>
                            </div>
                        </div>
                    </div>
                </section>
            </div>
        </main>

        <!-- Footer -->
        <footer class="bg-gray-800 text-white p-4 text-center mt-8">
            <p class="text-sm">
                © 2025 UlanBa?. All rights reserved.
            </p>
        </footer>
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