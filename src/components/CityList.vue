<template>
    <div v-for="city in savedCities" :key="city.id">
        <CityCard :city="city" @click="goToCityView(city)" />
    </div>

    <p v-if="savedCities.length === 0">
        No locations added. To start tracking a location, search in the field
        above.
    </p>
</template>

<script setup>
import { ref } from "vue";
import CityCard from "./CityCard.vue";
import { useRouter } from "vue-router";

const savedCities = ref([]);
const API_key = "358f3f60d3138ac781a219431c3d76b0";

const getCities = async () => {
    if (localStorage.getItem("savedCities")) {
        savedCities.value = JSON.parse(localStorage.getItem("savedCities"));

        const requests = savedCities.value.map((city) =>
            fetch(
                `https://api.openweathermap.org/data/2.5/weather?lat=${city.coords.lat}&lon=${city.coords.lon}&appid=${API_key}&units=metric`
            ).then((response) => {
                if (!response.ok) {
                    throw new Error(
                        `HTTP Error fetching data for ${city.name}`
                    );
                }
                return response.json();
            })
        );

        try {
            const weatherData = await Promise.all(requests);

            // Flicker delay
            await new Promise((res) => setTimeout(res, 1000))

            console.log("Les données récoltées: ", weatherData)

            weatherData.forEach((value, index) => {
                savedCities.value[index].weather = value;
            });
        } catch (error) {
            console.error("Error fetching weather data: ", error);
        }
    }
};

await getCities();

const router = useRouter();
const goToCityView = (city) => {
    router.push({
        name: "cityView",
        params: { city: city.city },
        query: {
            lat: city.coords.lat,
            lon: city.coords.lon,
            id: city.id, 
        },
    });
};
</script>
