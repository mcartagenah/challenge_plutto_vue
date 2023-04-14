<script setup>
import RutForm from './components/RutForm.vue'
</script>

<template>
  <header>
    <img alt="Vue logo" class="logo" src="./assets/plutto_logo.svg" width="250" height="250" />
  </header>

  <main>
    <RutForm />
  </main>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }
}
</style>
