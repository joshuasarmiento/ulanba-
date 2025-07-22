<template>
    <section class="mb-6">
        <!-- <h2 class="text-xl font-bold mb-3 text-gray-800">Saan ka?</h2> -->
        <div class="grid grid-cols-1 md:grid-cols-3 gap-2 md:gap-4">
            <Listbox :modelValue="selectedRegionCode" @update:modelValue="emit('update:selectedRegionCode', $event)">
                <div class="relative mt-1">
                    <ListboxButton
                        class="relative w-full cursor-default rounded-lg bg-white py-3 pl-3 pr-10 text-left shadow-sm focus:outline-none focus-visible:border-blue-500 focus-visible:ring-2 focus-visible:ring-white/75 focus-visible:ring-offset-2 focus-visible:ring-offset-blue-300 sm:text-sm">
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
                            class="absolute z-10 mt-1 max-h-60 w-full overflow-auto rounded-md bg-white py-1 text-base shadow-md ring-1 ring-black/5 focus:outline-none sm:text-sm">
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
                                        class="absolute inset-y-0 left-0 flex items-center pl-3 text-blue-600">
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
                        class="relative w-full cursor-default rounded-lg bg-white py-3 pl-3 pr-10 text-left shadow-sm focus:outline-none focus-visible:border-blue-500 focus-visible:ring-2 focus-visible:ring-white/75 focus-visible:ring-offset-2 focus-visible:ring-offset-blue-300 sm:text-sm">
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
                            class="absolute z-10 mt-1 max-h-60 w-full overflow-auto rounded-md bg-white py-1 text-base shadow-md ring-1 ring-black/5 focus:outline-none sm:text-sm">
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
                                        class="absolute inset-y-0 left-0 flex items-center pl-3 text-blue-600">
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

            <Listbox :modelValue="selectedCity" :disabled="!selectedProvinceCode" @update:modelValue="emit('update:selectedCity', $event)">
                <div class="relative mt-1">
                    <ListboxButton
                        class="relative w-full cursor-default rounded-lg bg-white py-3 pl-3 pr-10 text-left shadow-sm focus:outline-none focus-visible:border-blue-500 focus-visible:ring-2 focus-visible:ring-white/75 focus-visible:ring-offset-2 focus-visible:ring-offset-blue-300 sm:text-sm">
                        <span class="block truncate">{{ selectedCity ? selectedCity.name : 'Lungsod / Bayan' }}</span>
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
                            class="absolute z-10 mt-1 max-h-60 w-full overflow-auto rounded-md bg-white py-1 text-base shadow-md ring-1 ring-black/5 focus:outline-none sm:text-sm">
                            <ListboxOption v-slot="{ active, selected }" v-for="city in cities" :key="city.code"
                                :value="city" as="template">
                                <li :class="[
                                    active ? 'bg-blue-100 text-blue-900' : 'text-gray-900',
                                    'relative cursor-default select-none py-2 pl-10 pr-4',
                                ]">
                                    <span :class="[
                                        selected ? 'font-medium' : 'font-normal',
                                        'block truncate',
                                    ]">{{ city.name }}</span>
                                    <span v-if="selected"
                                        class="absolute inset-y-0 left-0 flex items-center pl-3 text-blue-600">
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
        </div>
    </section>
</template>

<script setup>
import {
    Listbox,
    ListboxButton,
    ListboxOptions,
    ListboxOption,
    TransitionRoot
} from '@headlessui/vue'

const props = defineProps({
    regions: Array,
    provinces: Array,
    cities: Array,
    selectedRegionCode: String,
    selectedProvinceCode: String,
    selectedCity: Object,
})

const emit = defineEmits([
    'update:selectedRegionCode',
    'update:selectedProvinceCode',
    'update:selectedCity',
])
</script>
