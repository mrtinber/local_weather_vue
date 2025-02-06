<template>
    <div class="flex flex-col flex-1">
        <!-- Weather Overview -->
        <div class="flex flex-col items-center py-12">
            <AnimatedPlaceholder class="max-w-[300px] h-[50px] w-full mb-12" />
            <AnimatedPlaceholder class="max-w-[300px] h-[100px] w-full mb-8" />
            <AnimatedPlaceholder class="max-w-[300px] w-full" />
            <AnimatedPlaceholder class="max-w-[300px] w-full mb-8" />
            <AnimatedPlaceholder class="max-w-[300px] h-[75px] w-full mb-8" />
            <AnimatedPlaceholder class="max-w-[300px] w-full" />
        </div>

        <hr class="border-white border-opacity-10 border w-full" />

        <!-- Daily -->
        <div class="flex flex-col items-center py-12 gap-12">
            <AnimatedPlaceholder
                class="max-w-screen-md h-[150px] w-full py-12"
            />
            <AnimatedPlaceholder
                class="max-w-screen-md h-[90px] w-full py-12"
            />
        </div>

        <div class="flex flex-col items-center">
            <AnimatedPlaceholder class="max-w-[300px] py-2 px-12" />
        </div>
    </div>
</template>

<script setup>
import AnimatedPlaceholder from "./AnimatedPlaceholder.vue";
</script>
