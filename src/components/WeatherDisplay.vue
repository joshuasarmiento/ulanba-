<template>
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
    <div v-if="!weatherData && !loading && !error && !selectedCity"
        class="text-center h-[60vh] p-10 flex flex-col items-center justify-center">
        <img src="https://raw.githubusercontent.com/Makin-Things/weather-icons/master/animated/dust.svg"
            alt="Searching Icon" class="w-24 h-auto mb-4" />
        <p class="text-gray-600 text-lg">Searching for cities... Please select a Lungsod / Bayan.</p>
    </div>

    <!-- Main Weather Data -->
    <div v-if="weatherData && !loading">
        <!-- Weather Alerts -->
        <section v-if="weatherData.alerts && weatherData.alerts.alert.length"
            class="mb-8 p-4 bg-white rounded-xl shadow">
            <h2 class="text-2xl font-bold mb-4 text-red-600">Weather Alerts</h2>
            <div v-for="(alert, index) in weatherData.alerts.alert" :key="index"
                class="bg-red-100 border-l-4 border-red-500 text-red-700 p-4 rounded-lg shadow mb-4 space-y-2">
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

        <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">

            <!-- Current Weather -->
            <div class="glow-box lg:col-span-1 bg-white p-8 rounded-xl shadow flex flex-col items-center text-center"
                data-glow>
                <h2 class="text-xl font-bold text-gray-600 mb-4">Panahon Ngayon sa <br> {{ selectedCity.name }}</h2>
                <p class="text-6xl font-extrabold text-gray-900">{{ weatherData.current.temp_c }}°C</p>
                <p class="text-gray-600 mb-2">Feels like: {{ weatherData.current.feelslike_c }}°C</p>
                <div class="flex items-center justify-center">
                    <img :src="getWeatherIcon(weatherData.current.condition.text, weatherData.current.is_day)"
                        alt="Weather Icon" class="w-32 h-auto " />
                    <div class="relative text-lg capitalize font-semibold text-gray-700">
                        <p class="text-sm"> {{ weatherData.current.condition.text }}</p>
                        <span :class="getWindWarningBadge(weatherData.current.wind_kph).color"
                            class="absolute -top-6 right-0 px-2 py-0.5 rounded-full text-[0.6rem] font-semibold flex items-center">
                            <span :class="getWindWarningBadge(weatherData.current.wind_kph).dotColor"
                                class="relative flex h-2 w-2">
                                <span class="animate-ping absolute inline-flex h-full w-full rounded-full opacity-75"
                                    :class="getWindWarningBadge(weatherData.current.wind_kph).dotColor"></span>
                                <span class="relative inline-flex rounded-full h-2 w-2"
                                    :class="getWindWarningBadge(weatherData.current.wind_kph).dotColor"></span>
                            </span>
                            <span class="ml-1 flex">{{ getWindWarningBadge(weatherData.current.wind_kph).label }}</span>
                        </span>
                    </div>
                </div>
                <div class="grid grid-cols-2 gap-y-2 w-full mt-4 text-xs text-gray-500">
                    <div class="flex items-center">
                        <svg class="w-6 h-6 mr-2 text-blue-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"
                            xmlns="http://www.w3.org/2000/svg">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"></path>
                        </svg>
                        <span>{{ weatherData.current.humidity }}% Humidity</span>
                    </div>
                    <div class="flex items-center">
                        <svg class="w-5 h-5 mr-2 text-blue-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"
                            xmlns="http://www.w3.org/2000/svg">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z">
                            </path>
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                d="M15 11a3 3 0 11-6 0 3 3 0 016 0z"></path>
                        </svg>
                        <span>{{ weatherData.current.wind_kph }} kph Wind </span>
                    </div>
                    <div class="flex items-center">
                        <svg class="w-5 h-5 mr-2 text-yellow-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"
                            xmlns="http://www.w3.org/2000/svg">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                d="M12 3v1m0 16v1m9-9h1M3 12H2m15.325-7.757l.707-.707M3.929 19.071l.707-.707m0-14.142l-.707-.707M19.071 19.071l-.707-.707M12 18a6 6 0 100-12 6 6 0 000 12z">
                            </path>
                        </svg>
                        <span>Sunrise: {{ weatherData.forecast.forecastday[0].astro.sunrise }}</span>
                    </div>
                    <div class="flex items-center">
                        <svg class="w-5 h-5 mr-2 text-purple-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"
                            xmlns="http://www.w3.org/2000/svg">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                d="M20.354 15.354A9 9 0 018.646 3.646 9.003 9.003 0 0012 21a9 9 0 008.354-5.646z">
                            </path>
                        </svg>
                        <span>Sunset: {{ weatherData.forecast.forecastday[0].astro.sunset }}</span>
                    </div>
                </div>
            </div>

            <!-- 3-Day Forecast -->
            <div class="lg:col-span-2 bg-white p-8 rounded-xl shadow">
                <h2 class="text-2xl font-bold text-gray-600 mb-4">3-Day Forecast</h2>
                <p class="text-sm text-gray-500 mb-4">Weather forecast for the next three days</p>
                <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                    <div v-for="day in weatherData.forecast.forecastday" :key="day.date"
                        class="text-center p-4 bg-gray-50 shadow rounded-lg">
                        <h3 class="font-bold text-gray-600">{{ formatDate(day.date) }}</h3>
                        <img :src="getWeatherIcon(day.day.condition.text, true)" alt="Weather Icon"
                            class="w-24 my-2 h-auto mx-auto" />
                        <p class="font-semibold text-lg">{{ day.day.maxtemp_c }}° / {{ day.day.mintemp_c }}°</p>

                        <div class="flex items-center justify-center mt-2 text-sm text-[#56A0EE]">
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
                <div class="mt-4 text-start text-xs text-gray-600">
                    <p>{{ nextDayWeatherTip }}</p>
                </div>
            </div>

            <!-- Hourly Forecast with Tabs -->
            <div class="lg:col-span-3 bg-white p-8 rounded-xl shadow mt-4">
                <h2 class="text-2xl font-bold text-gray-600 mb-4">Hourly Forecast</h2>
                <p class="text-sm text-gray-500 mb-4">Weather forecast for the next 24 hours</p>
                <TabGroup>
                    <TabList class="flex space-x-1 rounded-xl bg-gray-300/20 p-1 ">
                        <Tab v-for="day in weatherData.forecast.forecastday" :key="day.date" as="template"
                            v-slot="{ selected }">
                            <button :class="[
                                'w-full cursor-pointer rounded-lg py-2.5 text-sm font-medium leading-5',
                                'ring-white/60 ring-offset-2 ring-offset-blue-400 focus:outline-none focus:ring-2',
                                selected
                                    ? 'bg-white text-blue-700 shadow'
                                    : 'text-blue-400 hover:bg-white/[0.12] hover:text-blue',
                            ]">
                                {{ formatDate(day.date) }}
                            </button>
                        </Tab>
                    </TabList>

                    <TabPanels class="mt-2">
                        <TabPanel v-for="day in weatherData.forecast.forecastday" :key="day.date" :class="[
                            'rounded-xl bg-white p-3',
                            'ring-white/60 ring-offset-2 ring-offset-blue-400 focus:outline-none focus:ring-2',
                        ]">
                            <div class="overflow-x-auto">
                                <div class="flex space-x-4 pb-2">
                                    <div v-for="hour in day.hour" :key="hour.time_epoch"
                                        class="text-center p-3 bg-gray-50 shadow rounded-lg w-24 flex-shrink-0">
                                        <p class="text-sm text-gray-600">{{ formatTime(hour.time) }}</p>
                                        <img :src="getWeatherIcon(hour.condition.text, hour.is_day)" alt="Weather Icon"
                                            class="w-12 h-auto mx-auto my-1" />
                                        <p class="font-semibold text-md">{{ hour.temp_c }}°C</p>
                                        <p class="text-xs text-[#56A0EE]">{{ hour.chance_of_rain }}% Rain</p>
                                    </div>
                                </div>
                            </div>
                        </TabPanel>
                    </TabPanels>
                </TabGroup>
            </div>
        </div>

        <section v-if="pagasaData" class="mt-8">
            <div  class="bg-white p-6 shadow rounded-xl text-sm text-gray-600">
                <h2 class="text-2xl font-bold text-gray-600 mb-4">PAGASA Daily Weather Forecast</h2>
                <p v-if="pagasaData.issued_at" class="mb-2"><strong>Issued At:</strong> {{ pagasaData.issued_at }}</p>
                <p v-if="pagasaData.synopsis" class="mb-4"><strong>Synopsis:</strong> {{ pagasaData.synopsis }}</p>

                <Disclosure v-slot="{ open }">
                    <DisclosureButton
                        class="flex w-full justify-between rounded-lg bg-blue-100 px-4 py-2 text-left text-sm font-medium text-blue-900 hover:bg-blue-200 focus:outline-none focus-visible:ring focus-visible:ring-blue-500/75 mb-2">
                        <h3 class="font-bold text-gray-600">Forecast Weather Conditions</h3>
                        <svg :class="open ? 'rotate-180 transform' : ''" class="h-5 w-5 text-blue-500" fill="none"
                            viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" d="M19.5 8.25l-7.5 7.5-7.5-7.5" />
                        </svg>
                    </DisclosureButton>
                    <DisclosurePanel class="px-4 pb-2 pt-4 text-sm text-gray-500">
                        <div v-if="pagasaData.forecast_weather_conditions && pagasaData.forecast_weather_conditions.length"
                            class="overflow-x-auto mb-4">
                            <table class="min-w-full bg-white border border-gray-300">
                                <thead>
                                    <tr>
                                        <th class="py-2 px-4 border-b">Place</th>
                                        <th class="py-2 px-4 border-b">Weather Condition</th>
                                        <th class="py-2 px-4 border-b">Caused By</th>
                                        <th class="py-2 px-4 border-b">Impacts</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <tr v-for="(condition, index) in pagasaData.forecast_weather_conditions"
                                        :key="index">
                                        <td class="py-2 px-4 border-b">{{ condition.place }}</td>
                                        <td class="py-2 px-4 border-b">{{ condition.weather_condition }}</td>
                                        <td class="py-2 px-4 border-b">{{ condition.caused_by }}</td>
                                        <td class="py-2 px-4 border-b">{{ condition.impacts }}</td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </DisclosurePanel>
                </Disclosure>

                <Disclosure v-slot="{ open }">
                    <DisclosureButton
                        class="flex w-full justify-between rounded-lg bg-blue-100 px-4 py-2 text-left text-sm font-medium text-blue-900 hover:bg-blue-200 focus:outline-none focus-visible:ring focus-visible:ring-blue-500/75 mb-2">
                        <h3 class="font-bold text-gray-600">Forecast Wind and Coastal Water Conditions</h3>
                        <svg :class="open ? 'rotate-180 transform' : ''" class="h-5 w-5 text-blue-500" fill="none"
                            viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" d="M19.5 8.25l-7.5 7.5-7.5-7.5" />
                        </svg>
                    </DisclosureButton>
                    <DisclosurePanel class="px-4 pb-2 pt-4 text-sm text-gray-500">
                        <div v-if="pagasaData.forecast_wind_conditions && pagasaData.forecast_wind_conditions.length"
                            class="overflow-x-auto mb-4">
                            <table class="min-w-full bg-white border border-gray-300">
                                <thead>
                                    <tr>
                                        <th class="py-2 px-4 border-b">Place</th>
                                        <th class="py-2 px-4 border-b">Speed</th>
                                        <th class="py-2 px-4 border-b">Direction</th>
                                        <th class="py-2 px-4 border-b">Coastal Water</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <tr v-for="(wind, index) in pagasaData.forecast_wind_conditions" :key="index">
                                        <td class="py-2 px-4 border-b">{{ wind.place }}</td>
                                        <td class="py-2 px-4 border-b">{{ wind.speed }}</td>
                                        <td class="py-2 px-4 border-b">{{ wind.direction }}</td>
                                        <td class="py-2 px-4 border-b">{{ wind.coastal_water }}</td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </DisclosurePanel>
                </Disclosure>

                <Disclosure v-slot="{ open }">
                    <DisclosureButton
                        class="flex w-full justify-between rounded-lg bg-blue-100 px-4 py-2 text-left text-sm font-medium text-blue-900 hover:bg-blue-200 focus:outline-none focus-visible:ring focus-visible:ring-blue-500/75 mb-2">
                        <h3 class="font-bold text-gray-600">Temperature and Relative Humidity</h3>
                        <svg :class="open ? 'rotate-180 transform' : ''" class="h-5 w-5 text-blue-500" fill="none"
                            viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" d="M19.5 8.25l-7.5 7.5-7.5-7.5" />
                        </svg>
                    </DisclosureButton>
                    <DisclosurePanel class="px-4 pb-2 pt-4 text-sm text-gray-500">
                        <div v-if="pagasaData.temperature_humidity" class="overflow-x-auto mb-4">
                            <table class="min-w-full bg-white border border-gray-300">
                                <thead>
                                    <tr>
                                        <th class="py-2 px-4 border-b">Metric</th>
                                        <th class="py-2 px-4 border-b">Max Value</th>
                                        <th class="py-2 px-4 border-b">Max Time</th>
                                        <th class="py-2 px-4 border-b">Min Value</th>
                                        <th class="py-2 px-4 border-b">Min Time</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <tr v-for="(temp, metric) in pagasaData.temperature_humidity" :key="metric">
                                        <td class="py-2 px-4 border-b">{{ metric }}</td>
                                        <td class="py-2 px-4 border-b">{{ temp.max.value }}</td>
                                        <td class="py-2 px-4 border-b">{{ temp.max.time }}</td>
                                        <td class="py-2 px-4 border-b">{{ temp.min.value }}</td>
                                        <td class="py-2 px-4 border-b">{{ temp.min.time }}</td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </DisclosurePanel>
                </Disclosure>

                <Disclosure v-slot="{ open }">
                    <DisclosureButton
                        class="flex w-full justify-between rounded-lg bg-blue-100 px-4 py-2 text-left text-sm font-medium text-blue-900 hover:bg-blue-200 focus:outline-none focus-visible:ring focus-visible:ring-blue-500/75 mb-2">
                        <h3 class="font-bold text-gray-600">Tides and Astronomical Information</h3>
                        <svg :class="open ? 'rotate-180 transform' : ''" class="h-5 w-5 text-blue-500" fill="none"
                            viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" d="M19.5 8.25l-7.5 7.5-7.5-7.5" />
                        </svg>
                    </DisclosureButton>
                    <DisclosurePanel class="px-4 pb-2 pt-4 text-sm text-gray-500">
                        <div v-if="pagasaData.astronomical_information" class="mb-4">
                            <ul class="list-disc list-inside">
                                <li v-for="(value, key) in pagasaData.astronomical_information" :key="key">
                                    <strong>{{ key }}:</strong> {{ value }}
                                </li>
                            </ul>
                        </div>
                        <h4 class="font-bold text-gray-600 mb-2">Tidal Predictions</h4>
                        <div v-if="pagasaData.tidal_predictions && pagasaData.tidal_predictions.length"
                            class="overflow-x-auto">
                            <table class="min-w-full bg-white border border-gray-300">
                                <thead>
                                    <tr>
                                        <th class="py-2 px-4 border-b">Type</th>
                                        <th class="py-2 px-4 border-b">Value</th>
                                        <th class="py-2 px-4 border-b">Time</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <tr v-for="(tidal, index) in pagasaData.tidal_predictions" :key="index">
                                        <td class="py-2 px-4 border-b">{{ tidal.type }}</td>
                                        <td class="py-2 px-4 border-b">{{ tidal.value }}</td>
                                        <td class="py-2 px-4 border-b">{{ tidal.time }}</td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </DisclosurePanel>
                </Disclosure>
                <p class="text-sm text-gray-500 mt-4">
                    Data source:
                    <a href="https://www.pagasa.dost.gov.ph/weather#daily-weather-forecast" target="_blank"
                        rel="noopener noreferrer" class="text-blue-600 hover:text-blue-700 underline">
                        PAGASA - Philippine Atmospheric, Geophysical and Astronomical Services
                        Administration
                    </a>
                </p>
            </div>
        </section>
    </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import {
    Dialog,
    DialogPanel,
    DialogTitle,
    DialogDescription,
    TabGroup,
    TabList,
    Tab,
    TabPanels,
    TabPanel,
    Disclosure,
    DisclosureButton,
    DisclosurePanel,
} from '@headlessui/vue'

const props = defineProps({
    weatherData: Object,
    loading: Boolean,
    error: String,
    selectedCity: Object,
    getWeatherIcon: Function,
    formatDate: Function,
    formatDateTime: Function,
    formatTime: Function,
    pagasaData: Object,
})

const nextDayWeatherTip = computed(() => {
    if (!props.weatherData || !props.weatherData.forecast || !props.weatherData.forecast.forecastday[1]) {
        return "Keep an eye on the forecast!";
    }

    const tomorrow = props.weatherData.forecast.forecastday[1];
    const condition = tomorrow.day.condition.text.toLowerCase();
    const tips = [];

    if (condition.includes('sunny') || condition.includes('clear')) {
        tips.push("Bukas ay maganda ang panahon! Magsuot ng magaan na damit at mag-enjoy sa labas.");
        tips.push("Huwag kalimutan ang sunscreen para protektahan ang iyong balat.");
    } else if (condition.includes('rain') || condition.includes('drizzle') || condition.includes('shower')) {
        tips.push("May posibilidad ng pag-ulan bukas. Magdala ng payong o kapote.");
        tips.push("Mag-ingat sa pagmamaneho dahil maaaring madulas ang kalsada.");
    } else if (condition.includes('cloudy') || condition.includes('overcast')) {
        tips.push("Maulap ang panahon bukas. Magandang pagkakataon para sa mga outdoor activities.");
    } else if (condition.includes('thunder') || condition.includes('thundery')) {
        tips.push("May babala ng thunderstorms bukas. Manatili sa loob ng bahay kung posible.");
    }

    if (tips.length > 0) {
        return tips[Math.floor(Math.random() * tips.length)];
    }

    return "Laging maging handa sa anumang pagbabago ng panahon.";
});

const getWindWarningBadge = (wind_kph) => {
    let label = "Normal";
    let color = "bg-gray-200 text-gray-600";
    let dotColor = "bg-gray-500"; // Default dot color

    if (wind_kph >= 30 && wind_kph <= 60) {
        label = "PSWS #1";
        color = "bg-gray-500 text-white";
        dotColor = "bg-gray-500";
    } else if (wind_kph >= 61 && wind_kph <= 120) {
        label = "PSWS #2";
        color = "bg-blue-500 text-white";
        dotColor = "bg-blue-500";
    } else if (wind_kph >= 121 && wind_kph <= 170) {
        label = "PSWS #3";
        color = "bg-yellow-500 text-white";
        dotColor = "bg-yellow-500";
    } else if (wind_kph >= 171 && wind_kph <= 220) {
        label = "PSWS #4";
        color = "bg-orange-500 text-white";
        dotColor = "bg-orange-500";
    } else if (wind_kph > 220) {
        label = "PSWS #5";
        color = "bg-red-500 text-white";
        dotColor = "bg-red-500";
    }

    return { label, color, dotColor };
};

onMounted(() => {
    const glowBoxes = document.querySelectorAll('[data-glow]')

    glowBoxes.forEach(box => {
        box.addEventListener('mousemove', (e) => {
            const { x, y } = box.getBoundingClientRect()
            box.style.setProperty('--x', e.clientX - x)
            box.style.setProperty('--y', e.clientY - y)
        })
    })
})
</script>

<style scoped>
@keyframes ping {
    0% {
        transform: scale(1);
        opacity: 1;
    }

    75%,
    100% {
        transform: scale(2);
        opacity: 0;
    }
}

.animate-ping {
    animation: ping 1s cubic-bezier(0, 0, 0.2, 1) infinite;
}

@media screen and (min-width: 768px) {
    :root {
        --backdrop: hsla(0, 0%, 100%, 0.12);
        --radius: 2.075;
        --border: 2;
        --backup-border: var(--backdrop);
        --size: 400;
    }

    .glow-box:first-of-type {
        /* Changed to .glow-box */
        --base: 80;
        --spread: 500;
        --outer: 1;
    }

    .glow-box:last-of-type {
        /* Changed to .glow-box */
        --outer: 1;
        --base: 220;
        --spread: 200;
    }

    .glow-box {
        /* Changed to .glow-box */
        border-radius: calc(var(--radius) * 2rem);
        /* position: relative; */
        /* grid-template-rows: 1fr auto; */
        /* box-shad 0 1rem 2rem -1rem black; */
        backdrop-filter: blur(calc(var(--cardblur, 5) * 1px));
    }

    /* Glow specific styles */
    [data-glow] {
        --border-size: calc(var(--border, 2) * 1px);
        --spotlight-size: calc(var(--size, 150) * 1px);
        --hue: calc(var(--base) + (var(--xp, 0) * var(--spread, 0)));
        background-image: radial-gradient(var(--spotlight-size) var(--spotlight-size) at calc(var(--x, 0) * 1px) calc(var(--y, 0) * 1px),
                hsl(var(--hue, 210) calc(var(--saturation, 100) * 1%) calc(var(--lightness, 70) * 1%) / var(--bg-spot-opacity, 0.1)), transparent);
        background-color: var(--backdrop, transparent);
        background-size: calc(100% + (2 * var(--border-size))) calc(100% + (2 * var(--border-size)));
        background-position: 50% 50%;
        background-attachment: fixed;
        border: var(--border-size) solid var(--backup-border);
        position: relative;
        touch-action: none;
    }

    [data-glow]::before,
    [data-glow]::after {
        pointer-events: none;
        content: "";
        position: absolute;
        inset: calc(var(--border-size) * -1);
        border: var(--border-size) solid transparent;
        border-radius: calc(var(--radius) * 2rem);
        background-attachment: fixed;
        background-size: calc(100% + (2 * var(--border-size))) calc(100% + (2 * var(--border-size)));
        background-repeat: no-repeat;
        background-position: 50% 50%;
        mask:
            linear-gradient(transparent, transparent),
            linear-gradient(white, white);
        mask-clip: padding-box, border-box;
        mask-composite: intersect;
    }

    /* This is the emphasis light */
    [data-glow]::before {
        background-image: radial-gradient(calc(var(--spotlight-size) * 0.75) calc(var(--spotlight-size) * 0.75) at calc(var(--x, 0) * 1px) calc(var(--y, 0) * 1px),
                hsl(var(--hue, 210) calc(var(--saturation, 100) * 1%) calc(var(--lightness, 50) * 1%) / var(--border-spot-opacity, 1)), transparent 100%);
        filter: brightness(2);
    }

    /* This is the spotlight */
    [data-glow]::after {
        background-image: radial-gradient(calc(var(--spotlight-size) * 0.5) calc(var(--spotlight-size) * 0.5) at calc(var(--x, 0) * 1px) calc(var(--y, 0) * 1px),
                hsl(0 100% 100% / var(--border-light-opacity, 1)), transparent 100%);
    }

    [data-glow] [data-glow] {
        position: absolute;
        inset: 0;
        will-change: filter;
        opacity: var(--outer, 1);
    }

    [data-glow]>[data-glow] {
        border-radius: calc(var(--radius) * 2rem);
        border-width: calc(var(--border-size) * 20);
        filter: blur(calc(var(--border-size) * 10));
        background: none;
        pointer-events: none;
    }

    [data-glow]>[data-glow]::before {
        inset: -10px;
        border-width: 10px;
    }

    [data-glow] [data-glow] {
        border: none;
    }

}
</style>