<script setup>
import CityCardSkeleton from "@/components/CityCardSkeleton.vue";
import CityList from "@/components/CityList.vue";
import { ref } from "vue";
import { useRouter } from "vue-router";

const searchQuery = ref("");
const queryTimeOut = ref(null);
const searchResults = ref(null);
const searchError = ref(null);

const router = useRouter();

const getSearchResults = () => {
    queryTimeOut.value = setTimeout(async () => {
        clearTimeout(queryTimeOut.value);
        if (searchQuery !== "") {
            try {
                const url = `https://nominatim.openstreetmap.org/search?q=${encodeURIComponent(
                    searchQuery.value
                )}&format=json&addressdetails=1&limit=5&accept-language=en`;
                const response = await fetch(url);
                const data = await response.json();
                searchResults.value = data.map((result) => ({
                    name: result.display_name, // Nom complet du lieu
                    lat: result.lat,
                    lon: result.lon,
                }));
                console.log(searchResults);
            } catch (error) {
                console.error(
                    "Erreur lors de la récupération de résultats: ",
                    error
                );
                searchError.value = true;
                searchResults.value = null;
            }
        } else {
            searchResults.value = null;
        }
    }, 1000);
};

const previewCity = (searchResult) => {
    console.log(searchResult);
    router.push({
        name: "cityView",
        params: { city: searchResult.name },
        query: {
            lat: searchResult.lat,
            lon: searchResult.lon,
            preview: true,
        },
    });
};
</script>

<template>
    <main class="container text-white">
        <div class="pt-4 mb-8 relative">
            <input
                type="text"
                v-model="searchQuery"
                @input="getSearchResults"
                placeholder="Search for a city or state..."
                class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
            />
            <ul
                class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
                v-if="searchResults"
            >
                <p v-if="searchError">
                    Sorry, something went wrong. Please try again.
                </p>
                <p v-if="!searchError && searchResults.length === 0">
                    No match for your query.
                </p>
                <template v-else>
                    <li
                        v-for="result in searchResults"
                        :key="result.lat + '-' + result.lon"
                        class="py-2 px-1 cursor-pointer flex justify-between"
                        @click="previewCity(result)"
                    >
                        <p>{{ result.name }}</p>
                        <p>{{ result.lat }} , {{ result.lon }}</p>
                    </li>
                </template>
            </ul>
        </div>
        <div class="flex flex-col gap-4">
            <Suspense>
                <CityList />
                <template #fallback>
                    <CityCardSkeleton />
                </template>
            </Suspense>
        </div>
    </main>
</template>
