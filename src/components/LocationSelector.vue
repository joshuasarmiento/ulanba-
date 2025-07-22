<template>
    <section class="mb-6">
        <!-- Favorite Cities -->
        <div v-if="favorites.length > 0" class="flex flex-wrap items-start justify-end gap-2">
            <h3 class="font-semibold text-sm text-gray-600 mb-2">Mga Paboritong Lungsod:</h3>
            <div class="flex flex-wrap gap-2">
                <span v-for="favCity in favorites" :key="favCity.name"
                    class="inline-flex items-center rounded-md bg-blue-50 px-2 py-1 text-xs font-medium text-[#56A0EE] ring-1 ring-inset ring-[#56A0EE]/10 cursor-pointer"
                    @click="emit('update:selectedCity', favCity)">
                    {{ favCity.name }}
                    <button type="button" class="ml-1 -mr-0.5 h-3.5 w-3.5 rounded-full inline-flex items-center justify-center text-[#56A0EE] hover:bg-blue-200"
                        @click.stop="emit('remove-favorite', favCity)">
                        <span class="sr-only">Remove {{ favCity.name }}</span>
                        <svg class="h-3.5 w-3.5" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
                            <path d="M6.28 5.22a.75.75 0 00-1.06 1.06L8.94 10l-3.72 3.72a.75.75 0 101.06 1.06L10 11.06l3.72 3.72a.75.75 0 101.06-1.06L11.06 10l3.72-3.72a.75.75 0 00-1.06-1.06L10 8.94 6.28 5.22z" />
                        </svg>
                    </button>
                </span>
            </div>
        </div>

        <div class="grid grid-cols-1 md:grid-cols-3 gap-2 md:gap-4 mt-4">
            <Listbox :modelValue="selectedRegionCode" @update:modelValue="emit('update:selectedRegionCode', $event)">
                <div class="relative mt-1">
                    <ListboxButton
                        class="relative w-full cursor-default rounded-lg bg-white py-3 pl-3 pr-10 text-left shadow focus:outline-none focus-visible:border-blue-500 focus-visible:ring-2 focus-visible:ring-white/75 focus-visible:ring-offset-2 focus-visible:ring-offset-blue-300 sm:text-sm">
                        <span class="block truncate">{{ selectedRegionCode ? regions.find(r => r.code === selectedRegionCode)?.name : 'Rehiyon' }}</span>
                        <span class="pointer-events-none absolute inset-y-0 right-0 flex items-center pr-2">
                            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor"
                                class="h-5 w-5 text-gray-400" aria-hidden="true">
                                <path fill-rule="evenodd"
                                    d="M10 3a.75.75 0 01.55.24l3.25 3.5a.75.75 0 11-1.1 1.02L10 4.852 7.3 7.76a.75.75 0 01-1.1-1.02l3.25-3.5A.75.75 0 0110 3zm-3.75 9.75a.75.75 0 011.1 1.02L10 15.148l2.7-2.908a.75.75 0 011.1 1.02l-3.25 3.5a.75.75 0 01-1.1 0l-3.25-3.5a.75.75 0 01.55-.24z"
                                    clip-rule="evenodd" />
                            </svg>
                        </span>
                    </ListboxButton>
                    <TransitionRoot leave="transition ease-in duration-100" leaveFrom="opacity-100" leaveTo="opacity-0">
                        <ListboxOptions
                            class="absolute z-10 mt-1 max-h-60 w-full overflow-auto rounded-md bg-white py-1 text-base shadow ring-1 ring-black/5 focus:outline-none sm:text-sm">
                            <ListboxOption v-slot="{ active, selected }" v-for="region in regions" :key="region.code"
                                :value="region.code" as="template">
                                <li :class="[
                                    active ? 'bg-blue-100 text-blue-900' : 'text-gray-900',
                                    'relative cursor-default select-none py-2 pl-10 pr-4',
                                ]">
                                    <span :class="[
                                        selected ? 'font-medium' : 'font-normal',
                                        'block truncate',
                                    ]">{{ region.name }}</span>
                                    <span v-if="selected"
                                        class="absolute inset-y-0 left-0 flex items-center pl-3 text-[#56A0EE]">
                                        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20"
                                            fill="currentColor" class="h-5 w-5" aria-hidden="true">
                                            <path fill-rule="evenodd"
                                                d="M16.704 4.153a.75.75 0 01.143 1.052l-8 10.5a.75.75 0 01-1.127.075l-4.5-4.5a.75.75 0 011.06-1.06l3.894 3.893 7.48-9.817a.75.75 0 011.05-.143z"
                                                clip-rule="evenodd" />
                                        </svg>
                                    </span>
                                </li>
                            </ListboxOption>
                        </ListboxOptions>
                    </TransitionRoot>
                </div>
            </Listbox>

            <Listbox :modelValue="selectedProvinceCode" :disabled="!selectedRegionCode" @update:modelValue="emit('update:selectedProvinceCode', $event)">
                <div class="relative mt-1">
                    <ListboxButton
                        class="relative w-full cursor-default rounded-lg bg-white py-3 pl-3 pr-10 text-left shadow focus:outline-none focus-visible:border-blue-500 focus-visible:ring-2 focus-visible:ring-white/75 focus-visible:ring-offset-2 focus-visible:ring-offset-blue-300 sm:text-sm">
                        <span class="block truncate">{{ selectedProvinceCode ? provinces.find(p => p.code === selectedProvinceCode)?.name : 'Probinsya' }}</span>
                        <span class="pointer-events-none absolute inset-y-0 right-0 flex items-center pr-2">
                            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor"
                                class="h-5 w-5 text-gray-400" aria-hidden="true">
                                <path fill-rule="evenodd"
                                    d="M10 3a.75.75 0 01.55.24l3.25 3.5a.75.75 0 11-1.1 1.02L10 4.852 7.3 7.76a.75.75 0 01-1.1-1.02l3.25-3.5A.75.75 0 0110 3zm-3.75 9.75a.75.75 0 011.1 1.02L10 15.148l2.7-2.908a.75.75 0 011.1 1.02l-3.25 3.5a.75.75 0 01-1.1 0l-3.25-3.5a.75.75 0 01.55-.24z"
                                    clip-rule="evenodd" />
                            </svg>
                        </span>
                    </ListboxButton>
                    <TransitionRoot leave="transition ease-in duration-100" leaveFrom="opacity-100" leaveTo="opacity-0">
                        <ListboxOptions
                            class="absolute z-10 mt-1 max-h-60 w-full overflow-auto rounded-md bg-white py-1 text-base shadow ring-1 ring-black/5 focus:outline-none sm:text-sm">
                            <ListboxOption v-slot="{ active, selected }" v-for="province in provinces"
                                :key="province.code" :value="province.code" as="template">
                                <li :class="[
                                    active ? 'bg-blue-100 text-blue-900' : 'text-gray-900',
                                    'relative cursor-default select-none py-2 pl-10 pr-4',
                                ]">
                                    <span :class="[
                                        selected ? 'font-medium' : 'font-normal',
                                        'block truncate',
                                    ]">{{ province.name }}</span>
                                    <span v-if="selected"
                                        class="absolute inset-y-0 left-0 flex items-center pl-3 text-[#56A0EE]">
                                        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20"
                                            fill="currentColor" class="h-5 w-5" aria-hidden="true">
                                            <path fill-rule="evenodd"
                                                d="M16.704 4.153a.75.75 0 01.143 1.052l-8 10.5a.75.75 0 01-1.127.075l-4.5-4.5a.75.75 0 011.06-1.06l3.894 3.893 7.48-9.817a.75.75 0 011.05-.143z"
                                                clip-rule="evenodd" />
                                        </svg>
                                    </span>
                                </li>
                            </ListboxOption>
                        </ListboxOptions>
                    </TransitionRoot>
                </div>
            </Listbox>

            <Combobox :modelValue="selectedCity" @update:modelValue="emit('update:selectedCity', $event)" :disabled="!selectedProvinceCode">
                <div class="relative mt-1">
                    <div
                        class="relative w-full cursor-default overflow-hidden rounded-lg bg-white text-left shadow focus:outline-none focus-visible:ring-2 focus-visible:ring-white/75 focus-visible:ring-offset-2 focus-visible:ring-offset-blue-300 sm:text-sm">
                        <ComboboxInput
                            class="w-full border-none rounded-lg py-3 pl-3 pr-10 text-sm leading-5 text-gray-900 focus:ring-0"
                            :displayValue="(city) => city?.name"
                            @change="query = $event.target.value"
                            :placeholder="selectedProvinceCode ? 'Lungsod / Bayan' : 'Pumili muna ng Probinsya'"
                        />
                        <!-- <span class="absolute inset-y-0 right-0 flex items-center pr-2">
                            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor"
                                class="h-5 w-5 text-gray-400" aria-hidden="true">
                                <path fill-rule="evenodd"
                                    d="M10 3a.75.75 0 01.55.24l3.25 3.5a.75.75 0 11-1.1 1.02L10 4.852 7.3 7.76a.75.75 0 01-1.1-1.02l3.25-3.5A.75.75 0 0110 3zm-3.75 9.75a.75.75 0 011.1 1.02L10 15.148l2.7-2.908a.75.75 0 011.1 1.02l-3.25 3.5a.75.75 0 01-1.1 0l-3.25-3.5a.75.75 0 01.55-.24z"
                                    clip-rule="evenodd" />
                            </svg>
                        </span> -->
                    </div>
                    <TransitionRoot leave="transition ease-in duration-100" leaveFrom="opacity-100" leaveTo="opacity-0"
                        @after-leave="query = ''">
                        <ComboboxOptions
                            class="absolute z-10 mt-1 max-h-60 w-full overflow-auto rounded-md bg-white py-1 text-base shadow ring-1 ring-black/5 focus:outline-none sm:text-sm">
                            <div v-if="filteredCities.length === 0 && query !== ''"
                                class="relative cursor-default select-none py-2 px-4 text-gray-700">
                                Walang nahanap.
                            </div>

                            <ComboboxOption v-for="city in filteredCities" :key="city.code" :value="city" as="template"
                                v-slot="{ selected, active }">
                                <li :class="[
                                    active ? 'bg-blue-100 text-blue-900' : 'text-gray-900',
                                    'relative cursor-default select-none py-2 pl-10 pr-4',
                                ]">
                                    <div class="flex items-center justify-between">
                                        <span :class="[
                                            selected ? 'font-medium' : 'font-normal',
                                            'block truncate',
                                        ]">{{ city.name }}</span>
                                        <button v-if="!favorites.some(fav => fav.name === city.name)"
                                            @click.stop="emit('add-favorite', city)"
                                            class="ml-2 p-1 rounded-full text-gray-400 hover:bg-gray-200 hover:text-blue-600 focus:outline-none focus-visible:ring-2 focus-visible:ring-blue-500/75"
                                            title="Add to Favorites">
                                            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor" class="w-5 h-5">
                                                <path d="M9.293 2.293a1 1 0 011.414 0l7 7A1 1 0 0117 11h-1v6a1 1 0 01-1 1h-2a1 1 0 01-1-1v-3a1 1 0 00-1-1H9a1 1 0 00-1 1v3a1 1 0 01-1 1H5a1 1 0 01-1-1v-6H3a1 1 0 01-.707-1.707l7-7z" />
                                            </svg>
                                        </button>
                                        <button v-else
                                            @click.stop="emit('remove-favorite', city)"
                                            class="ml-2 p-1 rounded-full text-red-400 hover:bg-red-200 hover:text-red-600 focus:outline-none focus-visible:ring-2 focus-visible:ring-red-500/75"
                                            title="Remove from Favorites">
                                            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor" class="w-5 h-5">
                                                <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.28 7.22a.75.75 0 00-1.06 1.06L8.94 10l-1.72 1.72a.75.75 0 101.06 1.06L10 11.06l1.72 1.72a.75.75 0 101.06-1.06L11.06 10l1.72-1.72a.75.75 0 00-1.06-1.06L10 8.94l-1.72-1.72z" clip-rule="evenodd" />
                                            </svg>
                                        </button>
                                    </div>
                                    <span v-if="selected"
                                        class="absolute inset-y-0 left-0 flex items-center pl-3 text-[#56A0EE]">
                                        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20"
                                            fill="currentColor" class="h-5 w-5" aria-hidden="true">
                                            <path fill-rule="evenodd"
                                                d="M16.704 4.153a.75.75 0 01.143 1.052l-8 10.5a.75.75 0 01-1.127.075l-4.5-4.5a.75.75 0 011.06-1.06l3.894 3.893 7.48-9.817a.75.75 0 011.05-.143z"
                                                clip-rule="evenodd" />
                                        </svg>
                                    </span>
                                </li>
                            </ComboboxOption>
                        </ComboboxOptions>
                    </TransitionRoot>
                </div>
            </Combobox>
        </div>
    </section>
</template>

<script setup>
import {
    Listbox,
    ListboxButton,
    ListboxOptions,
    ListboxOption,
    TransitionRoot,
    Combobox,
    ComboboxInput,
    ComboboxOptions,
    ComboboxOption,
} from '@headlessui/vue'
import { ref, computed, watch } from 'vue'
import axios from 'axios'

const props = defineProps({
    regions: Array,
    provinces: Array,
    selectedRegionCode: String,
    selectedProvinceCode: String,
    selectedCity: Object,
    favorites: Array,
})

const emit = defineEmits([
    'update:selectedRegionCode',
    'update:selectedProvinceCode',
    'update:selectedCity',
    'add-favorite',
    'remove-favorite',
])

const query = ref('')
const searchedCities = ref([])
const searchLoading = ref(false)
const searchError = ref(null)

const searchCities = async (searchQuery) => {
    if (searchQuery.length < 3) {
        searchedCities.value = []
        return
    }
    searchLoading.value = true
    searchError.value = null
    try {
        // This API endpoint allows searching across all cities/municipalities
        const response = await axios.get(`https://psgc.gitlab.io/api/cities-municipalities/?search=${searchQuery}`)
        searchedCities.value = response.data
    } catch (err) {
        searchError.value = 'Error searching cities.'
        console.error(err)
    } finally {
        searchLoading.value = false
    }
}

watch(query, (newQuery) => {
    searchCities(newQuery)
})

const filteredCities = computed(() =>
    query.value === ''
        ? searchedCities.value
        : searchedCities.value.filter((city) =>
            city.name
                .toLowerCase()
                .replace(/\s+/g, '')
                .includes(query.value.toLowerCase().replace(/\s+/g, ''))
        )
)
</script>
