<script setup lang="ts">
import type { ChatProps } from "../types/chatProps";

const props = defineProps<ChatProps>();

const formatTime = (timestamp: number): string => {
    const date = new Date(timestamp);
    const hours = String(date.getHours()).padStart(2, "0");
    const minutes = String(date.getMinutes()).padStart(2, "0");
    return `${hours}:${minutes}`;
};

const formattedTime = formatTime(props.timestamp);
</script>

<template>
    <div
        class="w-full flex gap-5 whitespace-pre-wrap"
        :style="{ justifyContent: isAi ? 'flex-start' : 'flex-end' }"
    >
        <img
            v-if="isAi"
            src="/aiface.svg"
            class="h-15 rounded-sm"
            style="box-shadow: 0px 4px 4px rgba(233, 233, 233, 0.25)"
        />
        <div class="flex gap-3 items-end">
            <p v-if="!isAi" class="text-xs text-gray-500">
                {{ formattedTime }}
            </p>
            <div
                class="p-4 bg-white font-medium flex items-center justify-center min-h-15 rounded-sm max-w-100 text-left break-all"
                style="box-shadow: 0px 4px 4px rgba(233, 233, 233, 0.25)"
            >
                {{ msg }}
            </div>
            <p v-if="isAi" class="text-xs text-gray-500">{{ formattedTime }}</p>
        </div>
    </div>
</template>
