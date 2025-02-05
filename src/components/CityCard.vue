<template>
    <div
        class="flex py-6 px-3 bg-weather-secondary rounded-md shadow-md cursor-pointer"
    >
        <div class="flex flex-col flex-1 gap-2">
            <h2 class="text-xl">{{ city.city }}</h2>
            <span class="text-xs">
                Lat: {{ city.coords.lat }} - Lon: {{ city.coords.lon }}
            </span>
        </div>
        <div class="flex flex-col gap-2">
            <p class="text-xl self-end">
                {{ Math.round(city.weather.main.temp) }}&deg;
            </p>
            <div class="flex gap-2">
                <span class="text-xs">
                    Hum: {{ Math.round(city.weather.main.humidity) }} %
                </span>
                <span class="text-xs">
                    Feels like: {{ Math.round(city.weather.main.feels_like) }}&deg;
                </span>
            </div>
        </div>
    </div>
</template>

<script setup>
defineProps({
    city: {
        type: Object,
        default: () => {},
    },
});
</script>
