<template>
    <div
        class="w-[25%] bg-weather-secondary rounded-md px-3 py-2 flex flex-col justify-between cursor-pointer hover:scale-105 transition-all"
    >
        <div class="flex items-center gap-2 opacity-60">
            <i :class="iconClass"></i>
            <p class="uppercase">{{ title }}</p>
        </div>
        <slot></slot>
    </div>
</template>

<script setup>
defineProps({
    title: {
        type: String,
    },
    iconClass: {
        type: String,
    },
});
</script>
