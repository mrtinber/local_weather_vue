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
            <h1 class="text-4xl mb-2">{{ route.params.city }}</h1>
            <p class="text-sm mb-12">
                {{
                    new Date().toLocaleDateString("fr", {
                        weekday: "short",
                        day: "2-digit",
                        month: "long",
                    })
                }}
                {{
                    new Date().toLocaleTimeString("fr", {
                        timeStyle: "short",
                    })
                }}
            </p>
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
                alt=""
            />
        </div>

        <hr class="border-white border-opacity-10 border w-full" />

        <!-- Daily -->
        <div class="max-w-screen-md w-full py-12">
            <div class="mx-8 text-white">
                <h2 class="py-2">Daily Weather</h2>
                <div class="flex gap-10">
                    <div
                        v-for="element in weatherDaily.list.slice(0, 8)"
                        :key="element.dt"
                        class="flex flex-col gap-1 items-center"
                    >
                        <p class="whitespace-nowrap text-md">
                            {{
                                new Date(
                                    element.currentTime
                                ).toLocaleTimeString("fr", {
                                    hour: "2-digit",
                                })
                            }}
                        </p>
                        <img
                            class="w-auto h-[50px] object-cover"
                            :src="`http://openweathermap.org/img/wn/${element.weather[0].icon}@2x.png`"
                            alt=""
                        />
                        <p class="text-xl">
                            {{ Math.round(element.main.temp) }}&deg;
                        </p>
                    </div>
                </div>
            </div>
        </div>

        <div
            @click="removeCity"
            class="flex items-center gap-2 py-2 px-12 text-white cursor-pointer duration-150 hover:bg-red-500"
        >
            <i class="fa-solid fa-trash"></i>
            <p>Remove city</p>
        </div>
    </div>
</template>

<script setup>
import { useRoute, useRouter } from "vue-router";

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

        // Calculate hourly weather offset
        weatherDaily.list.forEach((hour) => {
            const utc = hour.dt * 1000 + localOffset;
            hour.currentTime = utc + 1000 * weatherDaily.city.timezone;
        });

        await new Promise((res) => setTimeout(res, 1000));

        return { weatherCurrent, weatherDaily };
    } catch (error) {
        console.error(error);
    }
};

const { weatherCurrent, weatherDaily } = await getWeatherData();

const router = useRouter();
const removeCity = () => {
    const cities = JSON.parse(localStorage.getItem("savedCities"));
    const updatedCities = cities.filter((city) => city.id !== route.query.id);
    localStorage.setItem("savedCities", JSON.stringify(updatedCities));
    router.push({
        name: "home",
    });
};
</script>
