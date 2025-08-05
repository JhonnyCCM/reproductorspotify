<script setup>
defineProps({
  cancion: Object
});
</script>

<template>
  <div class="text-center pt-2">
    <h1 class="text-2xl font-semibold">{{ cancion.titulo }}</h1>
    <p class="text-sm opacity-60">{{ cancion.nombre }}</p>
  </div>
</template>