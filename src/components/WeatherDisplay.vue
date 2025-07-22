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
        <Dialog :open="isLpaAdvisoryOpen" @close="closeLpaAdvisory" class="relative z-50">
            <div class="fixed inset-0 bg-black/30" aria-hidden="true" />

            <div class="fixed inset-0 flex w-screen items-center justify-center p-4">
                <DialogPanel class="w-full max-w-2xl transform overflow-hidden rounded-2xl bg-white p-6 text-left align-middle shadow-xl transition-all">
                    <DialogTitle class="text-lg font-medium leading-6 mb-2 text-amber-800">LPA & Typhoon Advisory</DialogTitle>
                    <DialogDescription class="text-sm text-gray-600 mb-4">
                        <div class="bg-orange-100 border-l-4 border-orange-500 text-orange-700 p-4" role="alert">
                            <div class="font-semibold mb-2">Para sa pinakahuling update tungkol sa Low Pressure Area (LPA) o
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
                                <h3 class="font-semibold mb-1">Mga Dapat Tandaan:</h3>
                                <ul class="list-disc list-inside text-sm">
                                    <li>Manatiling updated sa ulat-panahon mula sa PAGASA.</li>
                                    <li>Ihanda ang inyong emergency go-bag.</li>
                                    <li>Iwasan ang mga lugar na peligroso sa baha at landslide.</li>
                                    <li>Maging alerto sa mga posibleng pagbabago ng panahon.</li>
                                </ul>
                            </div>
                        </div>
                    </DialogDescription>

                    <!-- <button @click="closeLpaAdvisory" class="mt-4 px-4 py-2 bg-blue-500 text-white rounded-md hover:bg-blue-600">
                        Got it!
                    </button> -->
                </DialogPanel>
            </div>
        </Dialog>

        <div  class="grid grid-cols-1 lg:grid-cols-3 gap-6">

            <!-- Current Weather -->
            <div class="glow-box lg:col-span-1 bg-white p-6 rounded-xl shadow-sm flex flex-col items-center text-center" data-glow>
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
            <div class="lg:col-span-2 bg-white p-6 rounded-xl shadow-sm" >
                <h2 class="text-xl font-bold text-gray-800 mb-4">3-Day Forecast</h2>
                <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                    <div v-for="day in weatherData.forecast.forecastday" :key="day.date"
                        class="text-center p-4 bg-gray-50 shadow-sm rounded-lg">
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
            <div class="bg-white p-4 shadow-sm rounded-xl text-sm text-gray-600">
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
            <div class="bg-white p-6 shadow-sm rounded-xl">
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
</template>

<script setup>
import { ref, onMounted } from 'vue'
import {
    Dialog,
    DialogPanel,
    DialogTitle,
    DialogDescription,
} from '@headlessui/vue'

const props = defineProps({
    weatherData: Object,
    loading: Boolean,
    error: String,
    selectedCity: Object,
    selectedRegionCode: String, // Added for the "Searching for cities..." message
    getWeatherIcon: Function,
    formatDate: Function,
    formatDateTime: Function,
    getWeatherAdvice: Function,
})

const isLpaAdvisoryOpen = ref(true)

function closeLpaAdvisory() {
    isLpaAdvisoryOpen.value = false
}

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
@media screen and (min-width: 768px) {
    :root {
        --backdrop: hsla(0, 0%, 100%, 0.12);
        --radius: 2.075;
        --border: 2;
        --backup-border: var(--backdrop);
        --size: 400;
    }
    
    .glow-box:first-of-type { /* Changed to .glow-box */
        --base: 80;
        --spread: 500;
        --outer: 1;
    }
    .glow-box:last-of-type { /* Changed to .glow-box */
        --outer: 1;
        --base: 220;
        --spread: 200;
    }
    .glow-box { /* Changed to .glow-box */
        border-radius: calc(var(--radius) * 2rem);
        /* position: relative; */
        /* grid-template-rows: 1fr auto; */
        /* box-shadow: 0 1rem 2rem -1rem black; */
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