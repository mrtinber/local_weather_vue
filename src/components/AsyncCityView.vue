<template>
    <div class="flex flex-col flex-1 items-center">
        <!-- Banner -->
        <div
            v-if="route.query.preview"
            class="text-white p-4 text-center w-full bg-weather-secondary"
        >
            <p>
                You are currently previewing this city, click the "+" icon to
                start tracking this city
            </p>
        </div>
        <!-- Weather Overview -->
        <div class="flex flex-col items-center text-white py-12">
            <h1 class="text-4xl mb-12">{{ weatherCurrent.name }}</h1>
            <p class="text-8xl mb-8">
                {{ Math.round(weatherCurrent.main.temp) }}&deg;
            </p>
            <p>
                Feels like
                {{ Math.round(weatherCurrent.main.feels_like) }}&deg;
            </p>
            <p class="capitalize">
                {{ weatherCurrent.weather[0].description }}
            </p>
            <img
                class="w-[150px] h-auto"
                :src="`http://openweathermap.org/img/wn/${weatherCurrent.weather[0].icon}@2x.png`"
                :alt="`${weatherCurrent.weather[0].description}`"
            />
            <p class="text-sm italic">
                Last update:
                {{ formatDate(weatherCurrent.currentTime) }}
                &ndash;
                {{ formatTime(weatherCurrent.currentTime) }}
                (local time)
            </p>
        </div>

        <hr class="border-white border-opacity-10 border w-full" />

        <!-- Daily -->
        <div class="max-w-screen-md w-full py-12">
            <div class="mx-8 text-white">
                <h2 class="text-xl font-medium py-2">24-hour forecast</h2>
                <div class="flex justify-between">
                    <div
                        v-for="element in weatherDaily.list.slice(0, 8)"
                        :key="element.dt"
                        class="flex flex-col gap-1 items-center"
                    >
                        <p class="whitespace-nowrap text-md">
                            {{
                                new Date(
                                    element.currentTime
                                ).toLocaleTimeString("en-US", {
                                    hour: "2-digit",
                                })
                            }}
                        </p>
                        <img
                            class="w-auto h-[50px] object-cover"
                            :src="`http://openweathermap.org/img/wn/${element.weather[0].icon}@2x.png`"
                            :alt="`${element.weather[0].description}`"
                        />
                        <p class="text-xl">
                            {{ Math.round(element.main.temp) }}&deg;
                        </p>
                    </div>
                </div>
            </div>
        </div>

        <!-- Other information -->
        <div class="max-w-screen-md w-full py-12 text-white">
            <div class="mx flex gap-4">
                <WeatherCard title="Humidity" iconClass="fa-solid fa-droplet">
                    <p class="text-3xl text-right">
                        {{ weatherCurrent.main.humidity }} %
                    </p>
                </WeatherCard>
                <WeatherCard
                    title="Pressure"
                    iconClass="fa-solid fa-gauge-high"
                >
                    <p class="text-3xl text-right">
                        {{ weatherCurrent.main.pressure }} hPa
                    </p>
                </WeatherCard>
                <WeatherCard title="Wind" iconClass="fa-solid fa-wind">
                    <div class="text-right">
                        <p class="text-3xl">
                            {{ weatherCurrent.wind.speed }} km/h
                        </p>
                        <p class="text-xs opacity-60">
                            Coming from:
                            {{ getWindDirection(weatherCurrent.wind.deg) }}
                        </p>
                    </div>
                </WeatherCard>
                <WeatherCard
                    :title="isDay ? 'Sunset' : 'Sunrise'"
                    iconClass="fa-solid fa-sun"
                >
                    <p
                        v-bind:class="{
                            'text-3xl': isNight,
                            'text-xs opacity-60': isDay,
                        }"
                        class="text-right text-nowrap"
                    >
                        <span v-if="isDay" class="text-xs">Sunrise:</span>
                        {{ formatTime(weatherCurrent.sys.sunrise) }}
                    </p>
                    <p
                        v-bind:class="{
                            'text-3xl': isDay,
                            'text-xs opacity-60': isNight,
                        }"
                        class="text-right text-nowrap"
                    >
                        <span v-if="isNight" class="text-xs">Sunset:</span>
                        {{ formatTime(weatherCurrent.sys.sunset) }}
                    </p>
                </WeatherCard>
            </div>
        </div>

        <DeleteBtn text="Remove city" />
    </div>
</template>

<script setup>
import { useRoute } from "vue-router";
import DeleteBtn from "./DeleteBtn.vue";
import WeatherCard from "./WeatherCard.vue";
import { computed, ref } from "vue";

const route = useRoute();

const getWeatherData = async () => {
    try {
        const API_key = "358f3f60d3138ac781a219431c3d76b0";
        const currentWeatherResponse = await fetch(
            `https://api.openweathermap.org/data/2.5/weather?lat=${route.query.lat}&lon=${route.query.lon}&appid=${API_key}&units=metric`
        );

        const dailyWeatherResponse = await fetch(
            `https://api.openweathermap.org/data/2.5/forecast?lat=${route.query.lat}&lon=${route.query.lon}&appid=${API_key}&units=metric`
        );

        if (!currentWeatherResponse.ok) {
            throw new Error(
                `HTTP Error fetching current weather! Status: ${currentWeatherResponse.status}`
            );
        }

        if (!dailyWeatherResponse.ok) {
            throw new Error(
                `HTTP Error fetching daily weather! Status: ${dailyWeatherResponse.status}`
            );
        }

        const weatherCurrent = await currentWeatherResponse.json();
        const weatherDaily = await dailyWeatherResponse.json();
        console.log("Data current: ", weatherCurrent);
        console.log("Data daily: ", weatherDaily);

        // Calculate current date & time offset
        const localOffset = new Date().getTimezoneOffset() * 60000;
        const utc = weatherCurrent.dt * 1000 + localOffset;
        weatherCurrent.currentTime = utc + 1000 * weatherCurrent.timezone;

        weatherCurrent.sys.sunrise =
            weatherCurrent.sys.sunrise * 1000 +
            localOffset +
            1000 * weatherCurrent.timezone;
        weatherCurrent.sys.sunset =
            weatherCurrent.sys.sunset * 1000 +
            localOffset +
            1000 * weatherCurrent.timezone;

        // Calculate hourly weather offset
        weatherDaily.list.forEach((hour) => {
            const utc = hour.dt * 1000 + localOffset;
            hour.currentTime = utc + 1000 * weatherDaily.city.timezone;
        });

        // Prevent flicker
        await new Promise((res) => setTimeout(res, 1000));

        console.log("Data updated: ", weatherCurrent);
        return { weatherCurrent, weatherDaily };
    } catch (error) {
        console.error(error);
    }
};

const { weatherCurrent, weatherDaily } = await getWeatherData();

// *** Handling daytime/nighttime with ref ***
// const isDay = ref(true);
// const isNight = ref(true);

// if (
//     weatherCurrent.currentTime <= weatherCurrent.sys.sunset &&
//     weatherCurrent.currentTime >= weatherCurrent.sys.sunrise
// ) {
//     isDay.value = true;
//     isNight.value = false;
// } else {
//     isDay.value = false;
//     isNight.value = true;
// }

// *** Handling daytime/nighttime with computed ***
const isDay = computed(
    () =>
        weatherCurrent.currentTime <= weatherCurrent.sys.sunset &&
        weatherCurrent.currentTime >= weatherCurrent.sys.sunrise
);

const isNight = computed(() => !isDay.value);

// Handling time format
const formatTime = (timestamp) => {
    return new Date(timestamp).toLocaleTimeString("en-US", {
        timeStyle: "short",
    });
};

// Handling date format
const formatDate = (timestamp) => {
    return new Date(timestamp).toLocaleDateString("en-US", {
        weekday: "short",
        day: "2-digit",
        month: "long",
    });
};

// Handling wind direction
const getWindDirection = (deg) => {
    if (deg === 0 || weatherCurrent.wind.deg === 360) return "N";
    if (deg === 90) return "E";
    if (deg === 180) return "S";
    if (deg === 270) return "W";
    if (deg > 270 && deg < 360) return "NW";
    if (deg > 180 && deg < 270) return "SW";
    if (deg > 90 && deg < 180) return "SE";
    return "NE";
};
</script>
