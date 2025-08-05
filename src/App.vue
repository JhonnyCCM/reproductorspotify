<script setup>
import { ref, onMounted, computed } from 'vue';

// --- ESTADO REACTIVO (antes eran variables globales) ---
const canciones = ref([]);
const indiceCancionActual = ref(0);
const modoAleatorio = ref(false);
const modoRepetir = ref(false);
const estaReproduciendo = ref(false);
const progresoActual = ref(0);

// --- REFERENCIAS AL TEMPLATE (reemplaza document.getElementById) ---
const audioPlayer = ref(null); // Para acceder al <audio>

// --- DATOS COMPUTADOS (se actualizan solos cuando el estado cambia) ---
const cancionActual = computed(() => {
  if (canciones.value.length > 0) {
    return canciones.value[indiceCancionActual.value];
  }
  return { titulo: 'Sin canción', nombre: 'Agrega música a tu playlist', fuente: '' };
});

// --- MÉTODOS (reemplazan las funciones de app.js) ---

// Manejar cambio de fondo
function manejarCambioDeFondo(e) {
  const file = e.target.files[0];
  if (file) {
    const reader = new FileReader();
    reader.onload = (event) => {
      document.getElementById('app-container').style.backgroundImage = `url(${event.target.result})`;
    };
    reader.readAsDataURL(file);
  }
}

// Manejar la selección de archivos de música
function manejarSeleccionDeMusica(e) {
  const files = Array.from(e.target.files);
  const nuevasCanciones = files.map(file => {
    const url = URL.createObjectURL(file);
    const nombreArchivo = file.name.replace(/\.[^/.]+$/, "");
    return {
      titulo: nombreArchivo,
      nombre: 'Archivo local',
      fuente: url
    };
  });
  
  canciones.value = [...canciones.value, ...nuevasCanciones];

  if (canciones.value.length === nuevasCanciones.length && nuevasCanciones.length > 0) {
    cargarCancion();
    setTimeout(() => reproducirCancion(), 150);
  }
}

// Carga la información de la canción actual en el reproductor
function cargarCancion() {
  if (cancionActual.value && audioPlayer.value) {
    audioPlayer.value.src = cancionActual.value.fuente;
    audioPlayer.value.loop = modoRepetir.value;
    if(estaReproduciendo.value) {
        setTimeout(() => reproducirCancion(), 100);
    }
  }
}

function reproducirPausar() {
  if (!canciones.value.length) return;
  if (audioPlayer.value.paused) {
    reproducirCancion();
  } else {
    pausarCancion();
  }
}

function reproducirCancion() {
  if(!canciones.value.length) return;
  audioPlayer.value.play();
  estaReproduciendo.value = true;
}

function pausarCancion() {
  audioPlayer.value.pause();
  estaReproduciendo.value = false;
}

function actualizarProgreso() {
    progresoActual.value = audioPlayer.value.currentTime;
}

function setProgreso() {
  audioPlayer.value.currentTime = progresoActual.value;
}

function siguienteCancion() {
    if (canciones.value.length <= 1) return;
    if (modoAleatorio.value) {
        indiceCancionActual.value = obtenerIndiceAleatorio();
    } else {
        indiceCancionActual.value = (indiceCancionActual.value + 1) % canciones.value.length;
    }
    cargarCancion();
}

function cancionAnterior() {
    if (canciones.value.length <= 1) return;
    if (modoAleatorio.value) {
        indiceCancionActual.value = obtenerIndiceAleatorio();
    } else {
        indiceCancionActual.value = (indiceCancionActual.value - 1 + canciones.value.length) % canciones.value.length;
    }
    cargarCancion();
}

function tocarDesdePlaylist(index) {
    indiceCancionActual.value = index;
    cargarCancion();
}

function obtenerIndiceAleatorio() {
    if (canciones.value.length <= 1) return 0;
    let nuevoIndice;
    do {
        nuevoIndice = Math.floor(Math.random() * canciones.value.length);
    } while (nuevoIndice === indiceCancionActual.value);
    return nuevoIndice;
}

function toggleAleatorio() {
    modoAleatorio.value = !modoAleatorio.value;
    if(modoAleatorio.value) modoRepetir.value = false;
    audioPlayer.value.loop = modoRepetir.value;
}

function toggleRepetir() {
    modoRepetir.value = !modoRepetir.value;
    if(modoRepetir.value) modoAleatorio.value = false;
    audioPlayer.value.loop = modoRepetir.value;
}

onMounted(() => {
  audioPlayer.value.addEventListener('loadedmetadata', () => {
    progresoActual.max = audioPlayer.value.duration;
  });
});
</script>

<template>
  <div id="app-container" class="flex justify-center items-center min-h-screen w-full bg-cover bg-center" style="background-image: url(/BackGround.jpg);">
    <div class="absolute inset-0 bg-black/50 backdrop-blur-md z-10"></div>
    
    <section class="z-20 flex flex-col items-center text-gray-200 w-[380px] p-6 rounded-2xl bg-white/10 space-y-4">
      
      <div class="flex justify-between w-full gap-4">
          <label for="musica-input-vue" class="flex-1 text-center py-2 px-4 rounded-full cursor-pointer transition-colors bg-gray-500/30 hover:bg-gray-500/50">+ Playlist</label>
          <input @change="manejarSeleccionDeMusica" type="file" multiple accept="audio/*" class="hidden" id="musica-input-vue">
          
          <label for="fondo-input-vue" class="flex-1 text-center py-2 px-4 rounded-full cursor-pointer transition-colors bg-gray-500/30 hover:bg-gray-500/50">Background</label>
          <input @change="manejarCambioDeFondo" type="file" accept="image/*" class="hidden" id="fondo-input-vue">
      </div>

      <div v-if="canciones.length > 0" class="bg-gray-500/30 w-full max-h-48 overflow-y-auto p-2.5 rounded-lg">
          <h2 class="text-lg font-bold mb-2">Mi Playlist</h2>
          <ul>
              <li 
                v-for="(cancion, index) in canciones" 
                :key="cancion.fuente"
                @click="tocarDesdePlaylist(index)"
                class="p-2 my-1 rounded-md cursor-pointer transition-colors"
                :class="index === indiceCancionActual ? 'bg-white/40' : 'hover:bg-white/20'"
              >
                  {{ cancion.titulo }}
              </li>
          </ul>
      </div>

      <div class="text-center pt-2">
        <h1 class="text-2xl font-semibold">{{ cancionActual.titulo }}</h1>
        <p class="text-sm opacity-60">{{ cancionActual.nombre }}</p>
      </div>

      <audio 
        ref="audioPlayer" 
        @timeupdate="actualizarProgreso"
        @ended="siguienteCancion"
        class="hidden"
      ></audio>

      <input 
        type="range" 
        v-model="progresoActual" 
        @change="setProgreso"
        :max="audioPlayer && !isNaN(audioPlayer.duration) ? audioPlayer.duration : 100"
        class="range range-xs [--range-shdw:white] [--thumb-size:1rem]"
      />
      
      <div class="flex justify-center items-center space-x-2">
          <button @click="toggleAleatorio" class="btn btn-circle border-none text-white/80 transition-transform hover:scale-110" :class="modoAleatorio ? 'bg-white/40' : 'bg-gray-500/30 hover:bg-gray-500/50'">
            <i class="bi bi-shuffle text-2xl"></i>
          </button>
          
          <button @click="cancionAnterior" class="btn btn-circle border-none text-white/80 bg-gray-500/30 hover:bg-gray-500/50 transition-transform hover:scale-110">
            <i class="bi bi-rewind-fill text-2xl"></i>
          </button>
          
          <button @click="reproducirPausar" class="btn btn-circle btn-lg border-2 border-white/50 text-white bg-gray-500/40 hover:bg-gray-500/60 transition-transform hover:scale-110">
            <i class="bi text-4xl" :class="estaReproduciendo ? 'bi-pause-fill' : 'bi-play-fill'"></i>
          </button>
          
          <button @click="siguienteCancion" class="btn btn-circle border-none text-white/80 bg-gray-500/30 hover:bg-gray-500/50 transition-transform hover:scale-110">
            <i class="bi bi-fast-forward-fill text-2xl"></i>
          </button>

          <button @click="toggleRepetir" class="btn btn-circle border-none text-white/80 transition-transform hover:scale-110" :class="modoRepetir ? 'bg-white/40' : 'bg-gray-500/30 hover:bg-gray-500/50'">
            <i class="bi bi-repeat text-2xl"></i>
          </button>
      </div>
    </section>
  </div>
</template>