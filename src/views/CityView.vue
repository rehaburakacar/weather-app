<template>
  <div>
    <Suspense>
      <template #default>
        <RetrievedCityView />
      </template>
    </Suspense>
  </div>
</template>
<script setup>
import RetrievedCityView from "../components/RetrievedCityView.vue";
</script>
