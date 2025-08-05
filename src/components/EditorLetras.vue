<script setup>
import { ref } from 'vue';

// Este componente maneja su propio estado interno para los inputs
const textoLineaActual = ref('');

defineProps({
  letrasEnEdicion: Array,
  tiempoMarcado: Number,
  formatearTiempo: Function,
});

const emit = defineEmits(['marcar-tiempo', 'agregar-linea', 'eliminar-linea', 'guardar', 'limpiar', 'cerrar']);

function agregar() {
  emit('agregar-linea', textoLineaActual.value);
  textoLineaActual.value = ''; // Limpiar input después de emitir
}
</script>

<template>
  <div class="bg-black/60 w-full p-4 rounded-lg border-2 border-blue-400/50">
    <h3 class="text-center text-lg font-bold mb-4 text-blue-300">✏️ Editor de Letras de Karaoke</h3>
    
    <div class="space-y-3 mb-4">
      <div class="flex items-center space-x-2">
        <button @click="emit('marcar-tiempo')" class="btn btn-sm bg-blue-500/70 hover:bg-blue-500/90 border-none text-white">📍 Marcar Tiempo</button>
        <span class="text-blue-200 text-sm">{{ formatearTiempo(tiempoMarcado) }}</span>
      </div>
      <div class="flex space-x-2">
        <input 
          v-model="textoLineaActual" 
          @keyup.enter="agregar"
          placeholder="Escribe la línea de la letra aquí..."
          class="input input-sm flex-1 bg-white/10 border-white/30 text-white placeholder-white/50"
        />
        <button @click="agregar" class="btn btn-sm bg-green-500/70 hover:bg-green-500/90 border-none text-white">➕ Añadir</button>
      </div>
    </div>

    <div class="bg-black/40 max-h-32 overflow-y-auto p-2 rounded mb-3">
        <div v-if="letrasEnEdicion.length === 0" class="text-center text-white/50 text-sm">No hay letras añadidas.</div>
        <div v-for="(linea, index) in letrasEnEdicion" :key="index" class="flex items-center justify-between p-1 hover:bg-white/10 rounded text-sm">
            <span class="text-blue-300 w-12">{{ formatearTiempo(linea.tiempo) }}</span>
            <span class="flex-1 text-white mx-2">{{ linea.texto }}</span>
            <button @click="emit('eliminar-linea', index)" class="btn btn-xs bg-red-500/70 hover:bg-red-500/90 border-none text-white">🗑️</button>
        </div>
    </div>

    <div class="flex justify-center space-x-2">
        <button @click="emit('guardar')" class="btn btn-sm bg-green-600/70 hover:bg-green-600/90 border-none text-white">💾 Guardar</button>
        <button @click="emit('limpiar')" class="btn btn-sm bg-orange-500/70 hover:bg-orange-500/90 border-none text-white">🧹 Limpiar</button>
        <button @click="emit('cerrar')" class="btn btn-sm bg-gray-500/70 hover:bg-gray-500/90 border-none text-white">❌ Cerrar</button>
    </div>
  </div>
</template>