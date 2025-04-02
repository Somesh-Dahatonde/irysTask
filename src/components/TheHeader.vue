<template>
    <header class="bg-white shadow-md p-4">
        <div class="container mx-auto flex justify-between items-center">
            <div class="flex items-center space-x-4">
                <img src="../assets/logo.svg" alt="Irys logo" class="h-10" />
                <h1 class="text-2xl font-bold">Irys</h1>
            </div>

            <nav class="hidden md:flex space-x-6">
                <router-link to="/" class="text-gray-700 hover:text-blue-500">Home</router-link>
            </nav>
            <button @click="isMenuOpen = !isMenuOpen" class="md:hidden text-gray-700">
                <Menu :size="24" />
            </button>
        </div>
        <transition name="slide">
            <nav v-if="isMenuOpen" class="md:hidden bg-white shadow-lg absolute top-16 left-0 w-full p-4 space-y-4">
                <router-link to="/" class="block text-gray-700 hover:text-blue-500">Home</router-link>
            </nav>
        </transition>
    </header>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Menu } from "lucide-vue-next";

const isMenuOpen = ref(false);
</script>

<style scoped>
.slide-enter-active,
.slide-leave-active {
    transition: transform 0.3s ease-in-out, opacity 0.3s;
}

.slide-enter-from,
.slide-leave-to {
    transform: translateY(-10px);
    opacity: 0;
}
</style>