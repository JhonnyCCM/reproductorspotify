<script setup>
const emit = defineEmits(['seleccionar-musica', 'seleccionar-fondo']);
</script>

<template>
  <div class="flex justify-between w-full gap-4">
    <label for="musica-input-vue" class="flex-1 text-center py-2 px-4 rounded-full cursor-pointer transition-colors bg-gray-500/30 hover:bg-gray-500/50">+ Playlist</label>
    <input @change="emit('seleccionar-musica', $event)" type="file" multiple accept="audio/*" class="hidden" id="musica-input-vue">
    
    <label for="fondo-input-vue" class="flex-1 text-center py-2 px-4 rounded-full cursor-pointer transition-colors bg-gray-500/30 hover:bg-gray-500/50">Background</label>
    <input @change="emit('seleccionar-fondo', $event)" type="file" accept="image/*" class="hidden" id="fondo-input-vue">
  </div>
</template>