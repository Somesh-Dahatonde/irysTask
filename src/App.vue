<script setup>
import TheHeader from './components/TheHeader.vue'
import TheWelcome from "./components/TheWelcome.vue";
</script>


<template>
  <div class="app">
    <TheHeader />
    <main className="min-h-screen bg-gradient-to-br from-slate-50 to-slate-100 p-4 md:p-8">
      <TheWelcome />
    </main>
  </div>
</template>
