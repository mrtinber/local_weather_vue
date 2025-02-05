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
            <p class="text-sm mb-12 italic">
                Last update:
                {{
                    new Date(weatherCurrent.currentTime).toLocaleDateString(
                        "en-US",
                        {
                            weekday: "short",
                            day: "2-digit",
                            month: "long",
                        }
                    )
                }}
                -
                {{
                    new Date(weatherCurrent.currentTime).toLocaleTimeString(
                        "en-US",
                        {
                            timeStyle: "short",
                        }
                    )
                }}
                (local time)
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
                            alt=""
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
                <div class="w-[25%] bg-weather-secondary rounded-md px-3 py-2 flex flex-col justify-between">
                    <div class="flex items-center gap-2 opacity-60">
                        <i class="fa-solid fa-droplet"></i>
                        <p class="uppercase">Humidity</p>
                    </div>
                    <p class="text-3xl text-right">
                        {{ weatherCurrent.main.humidity }} %
                    </p>
                </div>
                <div class="w-[25%] bg-weather-secondary rounded-md px-3 py-2 flex flex-col justify-between">
                    <div class="flex items-center gap-2 opacity-60">
                        <i class="fa-solid fa-gauge-high"></i>
                        <p class="uppercase">Pressure</p>
                    </div>
                    <p class="text-3xl text-right">
                        {{ weatherCurrent.main.pressure }} hPa
                    </p>
                </div>
                <div class="w-[25%] bg-weather-secondary rounded-md px-3 py-2 flex flex-col justify-between">
                    <div class="flex items-center gap-2 opacity-60">
                        <i class="fa-solid fa-wind"></i>
                        <p class="uppercase">Wind</p>
                    </div>
                    <div>
                        <p class="text-3xl text-right">
                            {{ weatherCurrent.wind.speed }} km/h
                        </p>
                        <p class="text-xs text-right opacity-60">
                            Coming from:
                            {{
                                weatherCurrent.wind.deg === 0 ||
                                weatherCurrent.wind.deg === 360
                                    ? "N"
                                    : weatherCurrent.wind.deg === 90
                                    ? "E"
                                    : weatherCurrent.wind.deg === 180
                                    ? "S"
                                    : weatherCurrent.wind.deg === 270
                                    ? "W"
                                    : weatherCurrent.wind.deg > 270 &&
                                      weatherCurrent.wind.deg < 360
                                    ? "NW"
                                    : weatherCurrent.wind.deg > 180 &&
                                      weatherCurrent.wind.deg < 270
                                    ? "SW"
                                    : weatherCurrent.wind.deg > 90 &&
                                      weatherCurrent.wind.deg < 180
                                    ? "SE"
                                    : "NE"
                            }}
                        </p>
                    </div>
                </div>
                <div class="w-[25%] bg-weather-secondary rounded-md px-3 py-2">
                    <div class="flex items-center gap-2 opacity-60">
                        <i class="fa-solid fa-sun"></i>
                        <p class="uppercase"> {{ weatherCurrent.currentTime < weatherCurrent.sys.sunset * 1000 ? 'SUNSET' : "SUNRISE" }}</p>
                    </div>
                    <p v-bind:class="{ 'text-3xl': weatherCurrent.currentTime > weatherCurrent.sys.sunset * 1000, 'text-xs opacity-60': weatherCurrent.currentTime <= weatherCurrent.sys.sunset * 1000}" class="text-right text-nowrap">
                        <span v-if="weatherCurrent.currentTime <= weatherCurrent.sys.sunset * 1000" class="text-xs">Sunrise:</span>
                        {{
                            new Date(
                                weatherCurrent.sys.sunrise * 1000
                            ).toLocaleTimeString("en-US", {
                                timeStyle: "short",
                            })
                        }}
                    </p>
                    <p v-bind:class="{ 'text-3xl': weatherCurrent.currentTime < weatherCurrent.sys.sunset * 1000, 'text-xs opacity-60': weatherCurrent.currentTime >= weatherCurrent.sys.sunset * 1000}" class="text-right text-nowrap">
                        <span v-if="weatherCurrent.currentTime >= weatherCurrent.sys.sunset * 1000" class="text-xs">Sunset:</span>
                        {{
                            new Date(
                                weatherCurrent.sys.sunset * 1000
                            ).toLocaleTimeString("en-US", {
                                timeStyle: "short",
                            })
                        }}
                    </p>
                </div>
            </div>
        </div>

        <div
            @click="removeCity"
            class="flex items-center gap-2 py-2 px-12 text-white cursor-pointer duration-150 hover:bg-red-500 rounded-md hover:shadow-md mb-12"
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
