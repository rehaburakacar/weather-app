<template>
  <div class="flex flex-col min-h-screen bg-fixed bg-cover">
    <SiteNavigation />
    <RouterView class="flex-1" v-slot="{ Component }">
    </RouterView>
  </div>
</template>
<script setup>
import { RouterView } from "vue-router";
import SiteNavigation from "./components/SiteNavigation.vue";
</script>
<style>
.page-enter-active {
  transition: 600ms ease all;
}

.page-enter-from {
  opacity: 0;
}
</style>
