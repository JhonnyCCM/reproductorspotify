<script setup>
defineProps({
  cancion: {
    type: Object,
    // Añadimos un valor por defecto para prevenir errores si la canción es nula o indefinida
    default: () => ({ titulo: 'Sin canción', nombre: 'Agrega música a tu playlist' })
  }
});
</script>

<template>
  <div class="text-center pt-2">
    <h1 class="text-2xl font-semibold">{{ cancion?.titulo || 'Sin canción' }}</h1>
    <p class="text-sm opacity-60">{{ cancion?.nombre || 'Agrega música a tu playlist' }}</p>
  </div>
</template>