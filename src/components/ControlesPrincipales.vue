<script setup>
defineProps({
  estaReproduciendo: Boolean,
  modoAleatorio: Boolean,
  modoRepetir: Boolean,
  modoKaraoke: Boolean,
  modoEditor: Boolean,
});

const emit = defineEmits([
  'reproducir-pausar', 
  'anterior', 
  'siguiente', 
  'toggle-aleatorio', 
  'toggle-repetir',
  'toggle-karaoke',
  'toggle-editor'
]);
</script>

<template>
  <div class="flex justify-center items-center space-x-2">
    <button @click="emit('toggle-aleatorio')" class="btn btn-circle border-none text-white/80 transition-transform hover:scale-110" :class="modoAleatorio ? 'bg-white/40' : 'bg-gray-500/30 hover:bg-gray-500/50'">
        <i class="bi bi-shuffle text-2xl"></i>
    </button>
    <button @click="emit('anterior')" class="btn btn-circle border-none text-white/80 bg-gray-500/30 hover:bg-gray-500/50 transition-transform hover:scale-110">
        <i class="bi bi-rewind-fill text-2xl"></i>
    </button>
    <button @click="emit('reproducir-pausar')" class="btn btn-circle btn-lg border-2 border-white/50 text-white bg-gray-500/40 hover:bg-gray-500/60 transition-transform hover:scale-110">
        <i class="bi text-4xl" :class="estaReproduciendo ? 'bi-pause-fill' : 'bi-play-fill'"></i>
    </button>
    <button @click="emit('siguiente')" class="btn btn-circle border-none text-white/80 bg-gray-500/30 hover:bg-gray-500/50 transition-transform hover:scale-110">
        <i class="bi bi-fast-forward-fill text-2xl"></i>
    </button>
    <button @click="emit('toggle-repetir')" class="btn btn-circle border-none text-white/80 transition-transform hover:scale-110" :class="modoRepetir ? 'bg-white/40' : 'bg-gray-500/30 hover:bg-gray-500/50'">
        <i class="bi bi-repeat text-2xl"></i>
    </button>
    <button @click="emit('toggle-karaoke')" class="btn btn-circle border-none text-white/80 transition-transform hover:scale-110" :class="modoKaraoke ? 'bg-yellow-500/60 text-yellow-100' : 'bg-gray-500/30 hover:bg-gray-500/50'">
        <i class="bi bi-mic-fill text-2xl"></i>
    </button>
    <button @click="emit('toggle-editor')" class="btn btn-circle border-none text-white/80 transition-transform hover:scale-110" :class="modoEditor ? 'bg-blue-500/60 text-blue-100' : 'bg-gray-500/30 hover:bg-gray-500/50'">
        <i class="bi bi-pencil-fill text-2xl"></i>
    </button>
  </div>
</template>