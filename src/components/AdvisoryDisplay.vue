<template>
    <div v-if="alerts && alerts.alert && alerts.alert.length > 0" class="my-4">
        <div v-for="(alert, index) in alerts.alert" :key="index" class="p-4 rounded-lg" :class="alertClass(alert.severity)">
            <h3 class="font-bold text-lg mb-2">{{ alert.headline }}</h3>
            <p class="text-sm">{{ alert.desc }}</p>
            <p v-if="alert.instruction" class="text-sm mt-2"><strong>Instruction:</strong> {{ alert.instruction }}</p>
            <div class="text-xs mt-2 text-gray-700">
                <span>Effective: {{ formatDateTime(alert.effective) }}</span> |
                <span>Expires: {{ formatDateTime(alert.expires) }}</span>
            </div>
        </div>
    </div>
</template>

<script setup>
defineProps({
    alerts: Object,
});

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

const alertClass = (severity) => {
    const severityStr = severity.toLowerCase();
    if (severityStr.includes('moderate')) {
        return 'bg-yellow-100 border-l-4 border-yellow-500 text-yellow-700';
    }
    if (severityStr.includes('severe')) {
        return 'bg-red-100 border-l-4 border-red-500 text-red-700';
    }
    return 'bg-blue-100 border-l-4 border-blue-500 text-blue-700';
};
</script>
