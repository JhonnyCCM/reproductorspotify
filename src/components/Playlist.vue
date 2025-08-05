<script setup>
defineProps({
  canciones: Array,
  indiceCancionActual: Number
});

const emit = defineEmits(['seleccionar-cancion']);
</script>

<template>
  <div v-if="canciones.length > 0" class="bg-gray-500/30 w-full max-h-48 overflow-y-auto p-2.5 rounded-lg">
    <h2 class="text-lg font-bold mb-2">Mi Playlist</h2>
    <ul>
      <li 
        v-for="(cancion, index) in canciones" 
        :key="cancion.fuente"
        @click="emit('seleccionar-cancion', index)"
        class="p-2 my-1 rounded-md cursor-pointer transition-colors"
        :class="index === indiceCancionActual ? 'bg-white/40' : 'hover:bg-white/20'"
      >
        {{ cancion.titulo }}
      </li>
    </ul>
  </div>
</template>