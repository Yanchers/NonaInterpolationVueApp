<script setup lang="ts">
import { RouterView } from 'vue-router'
</script>

<template>
  <header class="m-3">
    <h1>Привет блин! Это маленькое приложения для интерполирования всяких штук...</h1>
  </header>

  <hr>

  <main class="m-3">
    <RouterView  />
  </main>
</template>

<style scoped>

</style>
