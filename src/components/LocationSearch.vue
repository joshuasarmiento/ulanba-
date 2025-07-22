<template>
    <div class="my-4">
        <div class="flex items-center w-full md:w-1/2 gap-2">
            <Combobox :modelValue="null" @update:modelValue="selectCity" as="div" class="relative flex-grow">
                <div
                    class="relative w-full cursor-default overflow-hidden rounded-lg bg-white text-left shadow focus:outline-none focus-visible:ring-2 focus-visible:ring-white/75 focus-visible:ring-offset-2 focus-visible:ring-offset-blue-300 sm:text-sm">
                    <ComboboxInput
                        class="w-full border-none rounded-lg py-2 pl-3 pr-10 text-sm leading-5 text-gray-900 focus:ring-2 focus:ring-blue-200"
                        :displayValue="(city) => city?.name" @change="searchQuery = $event.target.value"
                        :placeholder="props.selectedCity?.name || 'Search for a city...'" />
                    <ComboboxButton class="absolute inset-y-0 right-0 flex items-center pr-2">
                        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor"
                            class="h-5 w-5 text-gray-400" aria-hidden="true">
                            <path fill-rule="evenodd"
                                d="M10 3a.75.75 0 01.55.24l3.25 3.5a.75.75 0 11-1.1 1.02L10 4.852 7.3 7.76a.75.75 0 01-1.1-1.02l3.25-3.5A.75.75 0 0110 3zm-3.75 9.75a.75.75 0 011.1 1.02L10 15.148l2.7-2.908a.75.75 0 011.1 1.02l-3.25 3.5a.75.75 0 01-1.1 0l-3.25-3.5a.75.75 0 01.55-.24z"
                                clip-rule="evenodd" />
                        </svg>
                    </ComboboxButton>
                </div>
                <TransitionRoot leave="transition ease-in duration-100" leaveFrom="opacity-100" leaveTo="opacity-0"
                    @after-leave="searchQuery = ''">
                    <ComboboxOptions
                        class="absolute z-10 mt-1 max-h-60 w-full overflow-auto rounded-md bg-white py-1 text-base shadow ring-1 ring-black/5 focus:outline-none sm:text-sm">
                        <div v-if="(searchResults.length === 0 && searchQuery !== '') || (searchQuery === '' && metroManilaCities.length === 0)"
                            class="relative cursor-default select-none py-2 px-4 text-gray-700">
                            No results found.
                        </div>

                        <ComboboxOption v-for="result in searchResults" :key="result.id" :value="result" as="template"
                            v-slot="{ selected, active }">
                            <li :class="[
                                active ? 'bg-blue-100 text-blue-900' : 'text-gray-900',
                                'relative cursor-default select-none py-2 pl-10 pr-4',
                            ]">
                                <span :class="[
                                    selected ? 'font-medium' : 'font-normal',
                                    'block truncate',
                                ]">
                                    {{ result.name }}, {{ result.region }}, {{ result.country }}
                                </span>
                                <span v-if="selected"
                                    class="absolute inset-y-0 left-0 flex items-center pl-3 text-blue-600">
                                    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor"
                                        class="h-5 w-5" aria-hidden="true">
                                        <path fill-rule="evenodd"
                                            d="M16.704 4.153a.75.75 0 01.143 1.052l-8 10.5a.75.75 0 01-1.127.075l-4.5-4.5a.75.75 0 011.06-1.06l3.894 3.893 7.48-9.817a.75.75 0 011.05-.143z"
                                            clip-rule="evenodd" />
                                    </svg>
                                </span>
                            </li>
                        </ComboboxOption>
                    </ComboboxOptions>
                </TransitionRoot>
            </Combobox>
            <button @click="emit('use-my-location')"
                class="px-2 py-1.5 cursor-pointer text-white bg-[#CD9E73] md:bg-gradient-to-r md:from-[#CD9E73] md:to-[#56A0EE] rounded-lg shadow focus:outline-none focus:ring-2 focus:ring-blue-200">
                <svg class="w-6 h-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24"
                    xmlns="http://www.w3.org/2000/svg">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                        d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z">
                    </path>
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                        d="M15 11a3 3 0 11-6 0 3 3 0 016 0z"></path>
                </svg>
            </button>
        </div>
    </div>
</template>

<script setup>
import { ref, watch, watchEffect } from 'vue';
import axios from 'axios';
import {
    Combobox,
    ComboboxInput,
    ComboboxButton,
    ComboboxOptions,
    ComboboxOption,
    TransitionRoot,
} from '@headlessui/vue'

const emit = defineEmits(['city-selected', 'use-my-location']);

const props = defineProps({
    selectedCity: Object,
});

const searchQuery = ref('');
const searchResults = ref([]);
const metroManilaCities = ref([]);
const allPhilippineCities = ref([]);

let searchTimeout = null;

const fetchMetroManilaCities = async () => {
    try {
        const response = await axios.get(`https://psgc.gitlab.io/api/regions/130000000/cities-municipalities/`);
        metroManilaCities.value = response.data.map(city => ({
            id: city.code,
            name: city.name,
            region: 'Metro Manila',
            country: 'Philippines',
        }));
    } catch (error) {
        console.error('Error fetching Metro Manila cities:', error);
    }
};

const fetchAllPhilippineCities = async () => {
    try {
        const response = await axios.get(`https://psgc.gitlab.io/api/cities-municipalities/`);
        allPhilippineCities.value = response.data.map(city => ({
            id: city.code,
            name: city.name,
            region: city.region,
            country: 'Philippines',
        }));
    } catch (error) {
        console.error('Error fetching all Philippine cities:', error);
    }
};

watchEffect(() => {
    if (!metroManilaCities.value.length) {
        fetchMetroManilaCities();
    }
    if (!allPhilippineCities.value.length) {
        fetchAllPhilippineCities();
    }
    if (searchQuery.value === '') {
        searchResults.value = metroManilaCities.value;
    }
});

const onSearch = async () => {
    if (searchQuery.value === '') {
        searchResults.value = metroManilaCities.value;
        return;
    }

    if (searchQuery.value.length < 3) {
        searchResults.value = [];
        return;
    }

    if (searchTimeout) {
        clearTimeout(searchTimeout);
    }

    searchTimeout = setTimeout(() => {
        const filtered = allPhilippineCities.value.filter(city =>
            city.name.toLowerCase().includes(searchQuery.value.toLowerCase())
        );
        searchResults.value = filtered;
    }, 300);
};

const selectCity = (city) => {
    emit('city-selected', city);
    searchQuery.value = city.name;
    searchResults.value = [];
};

watch(searchQuery, onSearch);
</script>
