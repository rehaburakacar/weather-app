<template>
  <header class="sticky top-0 shadow-lg bg-sky-600 bg-opacity-20 backdrop-blur-lg z-50">
    <nav class="container flex flex-col  items-center gap-4 text-white py-6">
      <RouterLink :to="{ name: 'home' }">
        <div class="items-center">
          <p class="text-2xl">Vue Weather App</p>
        </div>
      </RouterLink>
    </nav>
  </header>
</template>
<script setup>
import { RouterLink } from "vue-router";
</script>
