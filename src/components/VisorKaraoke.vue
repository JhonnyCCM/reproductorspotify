<script setup>
import { computed, ref, watch } from 'vue';

const props = defineProps({
  letras: {
    type: Array,
    default: () => []
  },
  lineaActual: {
    type: Number,
    default: 0
  },
  opacidadVoz: {
    type: Number,
    default: 100
  }
});

const emit = defineEmits(['update:opacidadVoz']);

const opacidadModel = computed({
  get: () => props.opacidadVoz,
  set: (value) => emit('update:opacidadVoz', Number(value))
});

// Referencia al contenedor de las letras para auto-scroll
const letrasContainer = ref(null);

// Observador para hacer auto-scroll a la línea activa
watch(() => props.lineaActual, (newLinea) => {
  if (letrasContainer.value) {
    const activeElement = letrasContainer.value.querySelector('.karaoke-active');
    if (activeElement) {
      activeElement.scrollIntoView({
        behavior: 'smooth',
        block: 'center'
      });
    }
  }
});

</script>

<template>
  <div class="bg-black/40 w-full p-4 rounded-lg border-2 border-white/20 flex flex-col space-y-3">
    <h3 class="text-center text-lg font-bold text-white flex-shrink-0">🎤 Modo Karaoke 🎤</h3>
    
    <div ref="letrasContainer" class="flex-grow h-40 overflow-y-auto text-center space-y-2 pr-2">
      <p 
        v-for="(linea, index) in letras" 
        :key="index"
        class="transition-all duration-300 text-lg font-medium p-1"
        :class="{
          'text-yellow-300 text-xl scale-110 font-bold karaoke-active': index === lineaActual,
          'text-gray-400': index < lineaActual,
          'text-white/70': index > lineaActual
        }"
      >
        {{ linea.texto }}
      </p>
    </div>

    <div class="flex items-center space-x-3 w-full flex-shrink-0">
      <span class="text-sm text-white/80">🎵 Voz:</span>
      <input 
        type="range" 
        v-model="opacidadModel" 
        min="0" 
        max="100"
        class="range range-xs flex-1 [--range-shdw:yellow] [--thumb-size:0.8rem]"
      />
      <span class="text-sm text-white/60 w-8 text-center">{{ opacidadVoz }}%</span>
    </div>
  </div>
</template>