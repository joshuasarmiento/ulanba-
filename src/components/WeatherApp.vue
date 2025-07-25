<script setup>
import { ref, onMounted, watch, onUnmounted } from 'vue'
import axios from 'axios'
import AppHeader from './AppHeader.vue'
import AppFooter from './AppFooter.vue'
import LocationSearch from './LocationSearch.vue'
import WeatherDisplay from './WeatherDisplay.vue'
import AdvisoryDisplay from './AdvisoryDisplay.vue'

const selectedCity = ref(JSON.parse(localStorage.getItem('lastSelectedCity') || 'null'))

const favorites = ref(JSON.parse(localStorage.getItem('weatherFavorites') || '[]'))

const saveFavorites = () => {
    localStorage.setItem('weatherFavorites', JSON.stringify(favorites.value))
}

const addFavorite = (city) => {
    if (!favorites.value.some(fav => fav.name === city.name)) {
        favorites.value.push(city)
        saveFavorites()
    }
}

const removeFavorite = (city) => {
    favorites.value = favorites.value.filter(fav => fav.name !== city.name)
    saveFavorites()
}

const weatherData = ref(null)
const loading = ref(false)
const error = ref(null)
const alerts = ref(null)
const pagasaData = ref(null)
let advisoryInterval = null

const apiPagasa = import.meta.env.VITE_PAGASA_DAILY_FORECAST
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

const fetchWeatherByIp = () => {
    loading.value = true;
    error.value = null;

    if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
            async (position) => {
                const lat = position.coords.latitude;
                const lon = position.coords.longitude;
                try {
                    const geoResponse = await axios.get(`https://api.weatherapi.com/v1/current.json?key=${apiKey}&q=${lat},${lon}`);
                    const locationData = {
                        name: geoResponse.data.location.name,
                        region: geoResponse.data.location.region,
                        country: geoResponse.data.location.country
                    };
                    selectedCity.value = locationData;
                    fetchWeather();
                    // fetchPagasaData();
                } catch (apiError) {
                    error.value = 'Error fetching location details from WeatherAPI.';
                    console.error('WeatherAPI error:', apiError);
                    loading.value = false;
                }
            },
            (geoError) => {
                let errorMessage = 'Error getting your location.';
                switch (geoError.code) {
                    case geoError.PERMISSION_DENIED:
                        errorMessage = 'You denied access to your location. Please enable location services for this site.';
                        break;
                    case geoError.POSITION_UNAVAILABLE:
                        errorMessage = 'Your location is currently unavailable. Please check your internet connection or try again.';
                        break;
                    case geoError.TIMEOUT:
                        errorMessage = 'Timed out trying to get your location. Please try again.';
                        break;
                }
                error.value = errorMessage;
                console.error('Geolocation error:', geoError);
                loading.value = false;
            }
        );
    } else {
        error.value = 'Geolocation is not supported by your browser.';
        loading.value = false;
    }
};

const fetchPagasaData = async () => {
    loading.value = true
    error.value = null
    try {
        const response = await axios.get(`${apiPagasa}`)
        pagasaData.value = response.data
    } catch (err) {
        // More detailed error logging
        if (err.response) {
            // The request was made and the server responded with a status code
            // that falls out of the range of 2xx
            error.value = `Error fetching PAGASA data: Server responded with status ${err.response.status} - ${err.response.data.message || err.response.statusText}`;
            console.error('PAGASA API Error Response:', err.response.data);
        } else if (err.request) {
            // The request was made but no response was received
            // `error.request` is an instance of XMLHttpRequest in the browser and an instance of
            // http.ClientRequest in node.js
            error.value = 'Error fetching PAGASA data: No response received from server. Check if the API is running and accessible.';
            console.error('PAGASA API Request Error:', err.request);
        } else {
            // Something happened in setting up the request that triggered an Error
            error.value = `Error fetching PAGASA data: ${err.message}`;
            console.error('PAGASA API Axios Error:', err.message);
        }
        console.error("PAGASA API Error Response:", err.response.data); // Log the full error object for detailed debugging
    } finally {
        loading.value = false
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
        alerts.value = weatherResponse.data.alerts
    } catch (err) {
        error.value = 'Error: Hindi makuha ang weather data. Please try again later.'
        console.error(err)
    } finally {
        loading.value = false
    }
}

watch(selectedCity, (newCity) => {
    if (newCity) {
        fetchWeather()
        fetchPagasaData()
        localStorage.setItem('lastSelectedCity', JSON.stringify(newCity));
    } else {
        localStorage.removeItem('lastSelectedCity');
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
    const lastSelected = JSON.parse(localStorage.getItem('lastSelectedCity') || 'null');
    if (lastSelected) {
        selectedCity.value = lastSelected;
        fetchPagasaData();
    } else {
        fetchWeatherByIp();
    }

    advisoryInterval = setInterval(() => {
        fetchWeather();
        fetchPagasaData();
    }, 3600000) // 1 hour
})

onUnmounted(() => {
    clearInterval(advisoryInterval)
})
</script>

<template>
    <div class="min-h-screen bg-gradient-to-br from-blue-50 via-white to-green-50 font-sans">
        <main class="flex-grow p-4 max-w-5xl mx-auto w-full">
            <AppHeader :weatherData="weatherData" :getWeatherIcon="getWeatherIcon" />
            <LocationSearch :selectedCity="selectedCity" @city-selected="selectedCity = $event" @use-my-location="fetchWeatherByIp" />

            <AdvisoryDisplay :alerts="alerts" />

            <WeatherDisplay :weatherData="weatherData" :loading="loading" :error="error" :selectedCity="selectedCity"
                :selectedRegionCode="selectedRegionCode" :getWeatherIcon="getWeatherIcon" :formatDate="formatDate"
                :formatDateTime="formatDateTime" :formatTime="formatTime" :pagasaData="pagasaData" />
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