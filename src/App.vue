<template>
  <AppHeader />
  <main class="app-main">
    <RouterView />
  </main>
</template>

<script setup>
import AppHeader from '@/components/AppHeader.vue'
</script>

<style lang="scss">
.app-main {
  flex: 1;
  padding-bottom: $space-16;
}
</style>
