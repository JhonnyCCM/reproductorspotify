<script setup>
import { computed } from 'vue';

const props = defineProps({
  volumen: Number,
  estaSilenciado: Boolean,
});

const emit = defineEmits(['toggle-silencio', 'update:volumen']);

const volumenModel = computed({
  get: () => props.volumen,
  set: (value) => emit('update:volumen', value)
});
</script>

<template>
  <div class="flex items-center space-x-3 w-full">
    <button @click="emit('toggle-silencio')" class="btn btn-circle btn-sm border-none text-white/80 bg-gray-500/30 hover:bg-gray-500/50 transition-transform hover:scale-110">
      <i class="bi text-lg" :class="estaSilenciado || volumen == 0 ? 'bi-volume-mute-fill' : volumen < 50 ? 'bi-volume-down-fill' : 'bi-volume-up-fill'"></i>
    </button>
    <input 
      type="range" 
      v-model="volumenModel"
      min="0" 
      max="100"
      class="range range-xs flex-1 [--range-shdw:white] [--thumb-size:0.8rem]"
    />
    <span class="text-sm text-white/60 w-8 text-center">{{ volumen }}</span>
  </div>
</template>