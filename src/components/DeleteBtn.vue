<template>
    <div
        @click="removeCity"
        class="flex items-center gap-2 py-2 px-12 text-white cursor-pointer duration-150 hover:bg-red-500 rounded-md hover:shadow-md mb-12"
    >
        <i class="fa-solid fa-trash"></i>
        <p>{{ text }}</p>
    </div>
</template>

<script setup>
import { useRoute, useRouter } from "vue-router";

const route = useRoute();

const router = useRouter();
const removeCity = () => {
    const cities = JSON.parse(localStorage.getItem("savedCities"));
    const updatedCities = cities.filter((city) => city.id !== route.query.id);
    localStorage.setItem("savedCities", JSON.stringify(updatedCities));
    router.push({
        name: "home",
    });
};

defineProps({
    text: {
        type: String,
    },
});
</script>
