<script setup>
import { ref, onMounted, computed } from 'vue';

// --- ESTADO REACTIVO (antes eran variables globales) ---
const canciones = ref([]);
const indiceCancionActual = ref(0);
const modoAleatorio = ref(false);
const modoRepetir = ref(false);
const estaReproduciendo = ref(false);
const progresoActual = ref(0);
const volumenActual = ref(50);
const estaSilenciado = ref(false);
const volumenAnterior = ref(50);

// --- NUEVOS ESTADOS PARA KARAOKE ---
const modoKaraoke = ref(false);
const lineaActualKaraoke = ref(0);
const opacidadAudioOriginal = ref(100);

// --- ESTADOS PARA EDITOR DE LETRAS ---
const modoEditor = ref(false);
const textoLineaActual = ref('');
const letrasEnEdicion = ref([]);
const tiempoMarcado = ref(0);

// --- REFERENCIAS AL TEMPLATE ---
const audioPlayer = ref(null);

// --- DATOS DE LETRAS DE EJEMPLO ---
const letrasKaraoke = ref({
  // Ejemplo de letras con tiempos (en segundos)
  "default": [
    { tiempo: 0, texto: "♪ Bienvenido al modo karaoke ♪" },
    { tiempo: 3, texto: "Carga una canción para ver las letras aquí" },
    { tiempo: 6, texto: "¡Diviértete cantando!" },
    { tiempo: 10, texto: "🎤 ¡A cantar se ha dicho! 🎤" }
  ]
});

// --- DATOS COMPUTADOS ---
const cancionActual = computed(() => {
  if (canciones.value.length > 0) {
    return canciones.value[indiceCancionActual.value];
  }
  return { titulo: 'Sin canción', nombre: 'Agrega música a tu playlist', fuente: '' };
});

const letrasCancionActual = computed(() => {
  if (cancionActual.value && letrasKaraoke.value[cancionActual.value.titulo]) {
    return letrasKaraoke.value[cancionActual.value.titulo];
  }
  return letrasKaraoke.value.default;
});

// --- MÉTODOS EXISTENTES ---

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

function manejarSeleccionDeMusica(e) {
  const files = Array.from(e.target.files);
  const nuevasCanciones = files.map(file => {
    const url = URL.createObjectURL(file);
    const nombreArchivo = file.name.replace(/\.[^/.]+$/, "");
    
    // Añadir letras de ejemplo para cada canción
    letrasKaraoke.value[nombreArchivo] = [
      { tiempo: 5, texto: `♪ ${nombreArchivo} ♪` },
      { tiempo: 10, texto: "Esta es la primera línea de la canción" },
      { tiempo: 15, texto: "Aquí va el coro principal" },
      { tiempo: 20, texto: "Otra línea para cantar" },
      { tiempo: 25, texto: "¡Sigue el ritmo!" },
      { tiempo: 30, texto: "🎵 Final de la canción 🎵" }
    ];
    
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

function cargarCancion() {
  if (cancionActual.value && audioPlayer.value) {
    audioPlayer.value.src = cancionActual.value.fuente;
    audioPlayer.value.loop = modoRepetir.value;
    lineaActualKaraoke.value = 0; // Resetear línea de karaoke
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
  
  // Actualizar línea actual de karaoke
  if (modoKaraoke.value) {
    actualizarLineaKaraoke();
  }
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

function cambiarVolumen() {
    if (audioPlayer.value) {
        audioPlayer.value.volume = volumenActual.value / 100;
    }
}

function toggleSilencio() {
    if (estaSilenciado.value) {
        volumenActual.value = volumenAnterior.value;
        estaSilenciado.value = false;
    } else {
        volumenAnterior.value = volumenActual.value;
        volumenActual.value = 0;
        estaSilenciado.value = true;
    }
    cambiarVolumen();
}

// --- NUEVOS MÉTODOS PARA KARAOKE ---

function toggleKaraoke() {
  modoKaraoke.value = !modoKaraoke.value;
  if (modoKaraoke.value) {
    lineaActualKaraoke.value = 0;
    modoEditor.value = false; // Cerrar editor si está abierto
  }
}

function actualizarLineaKaraoke() {
  const tiempoActual = audioPlayer.value.currentTime;
  const letras = letrasCancionActual.value;
  
  for (let i = letras.length - 1; i >= 0; i--) {
    if (tiempoActual >= letras[i].tiempo) {
      lineaActualKaraoke.value = i;
      break;
    }
  }
}

function cambiarOpacidadAudio() {
  if (audioPlayer.value) {
    audioPlayer.value.volume = (volumenActual.value * opacidadAudioOriginal.value) / 10000;
  }
}

// --- MÉTODOS PARA EDITOR DE LETRAS ---

function toggleEditor() {
  modoEditor.value = !modoEditor.value;
  if (modoEditor.value) {
    modoKaraoke.value = false; // Cerrar karaoke si está abierto
    cargarLetrasExistentes();
  }
}

function cargarLetrasExistentes() {
  if (cancionActual.value && letrasKaraoke.value[cancionActual.value.titulo]) {
    letrasEnEdicion.value = [...letrasKaraoke.value[cancionActual.value.titulo]];
  } else {
    letrasEnEdicion.value = [];
  }
}

function marcarTiempo() {
  if (!audioPlayer.value) return;
  
  const tiempoActual = Math.round(audioPlayer.value.currentTime * 10) / 10; // Redondear a 1 decimal
  tiempoMarcado.value = tiempoActual;
}

function agregarLinea() {
  if (!textoLineaActual.value.trim()) return;
  
  const nuevaLinea = {
    tiempo: tiempoMarcado.value,
    texto: textoLineaActual.value.trim()
  };
  
  letrasEnEdicion.value.push(nuevaLinea);
  // Ordenar por tiempo
  letrasEnEdicion.value.sort((a, b) => a.tiempo - b.tiempo);
  
  // Limpiar input
  textoLineaActual.value = '';
}

function eliminarLinea(index) {
  letrasEnEdicion.value.splice(index, 1);
}

function guardarLetras() {
  if (!cancionActual.value) return;
  
  letrasKaraoke.value[cancionActual.value.titulo] = [...letrasEnEdicion.value];
  alert('¡Letras guardadas exitosamente!');
}

function limpiarEditor() {
  letrasEnEdicion.value = [];
  textoLineaActual.value = '';
  tiempoMarcado.value = 0;
}

function formatearTiempo(segundos) {
  const minutos = Math.floor(segundos / 60);
  const segs = segundos % 60;
  return `${minutos}:${segs.toFixed(1).padStart(4, '0')}`;
}

onMounted(() => {
  audioPlayer.value.addEventListener('loadedmetadata', () => {
    progresoActual.max = audioPlayer.value.duration;
  });
  
  audioPlayer.value.volume = volumenActual.value / 100;
});
</script>

<template>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.0/font/bootstrap-icons.css">

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

      <!-- EDITOR DE LETRAS -->
      <div v-if="modoEditor" class="bg-black/60 w-full p-4 rounded-lg border-2 border-blue-400/50">
        <h3 class="text-center text-lg font-bold mb-4 text-blue-300">✏️ Editor de Letras de Karaoke</h3>
        
        <!-- Controles del Editor -->
        <div class="space-y-3 mb-4">
          <div class="flex items-center space-x-2">
            <button @click="marcarTiempo" class="btn btn-sm bg-blue-500/70 hover:bg-blue-500/90 border-none text-white">
              📍 Marcar Tiempo
            </button>
            <span class="text-blue-200 text-sm">{{ formatearTiempo(tiempoMarcado) }}</span>
          </div>
          
          <div class="flex space-x-2">
            <input 
              v-model="textoLineaActual" 
              @keyup.enter="agregarLinea"
              placeholder="Escribe la línea de la letra aquí..."
              class="input input-sm flex-1 bg-white/10 border-white/30 text-white placeholder-white/50"
            />
            <button @click="agregarLinea" class="btn btn-sm bg-green-500/70 hover:bg-green-500/90 border-none text-white">
              ➕ Añadir
            </button>
          </div>
        </div>

        <!-- Lista de Letras en Edición -->
        <div class="bg-black/40 max-h-32 overflow-y-auto p-2 rounded mb-3">
          <div v-if="letrasEnEdicion.length === 0" class="text-center text-white/50 text-sm">
            No hay letras añadidas. ¡Empieza agregando algunas!
          </div>
          <div v-for="(linea, index) in letrasEnEdicion" :key="index" class="flex items-center justify-between p-1 hover:bg-white/10 rounded text-sm">
            <span class="text-blue-300 w-12">{{ formatearTiempo(linea.tiempo) }}</span>
            <span class="flex-1 text-white mx-2">{{ linea.texto }}</span>
            <button @click="eliminarLinea(index)" class="btn btn-xs bg-red-500/70 hover:bg-red-500/90 border-none text-white">
              🗑️
            </button>
          </div>
        </div>

        <!-- Botones de Acción -->
        <div class="flex justify-center space-x-2">
          <button @click="guardarLetras" class="btn btn-sm bg-green-600/70 hover:bg-green-600/90 border-none text-white">
            💾 Guardar
          </button>
          <button @click="limpiarEditor" class="btn btn-sm bg-orange-500/70 hover:bg-orange-500/90 border-none text-white">
            🧹 Limpiar
          </button>
          <button @click="toggleEditor" class="btn btn-sm bg-gray-500/70 hover:bg-gray-500/90 border-none text-white">
            ❌ Cerrar
          </button>
        </div>
      </div>
      <div v-if="modoKaraoke" class="bg-black/40 w-full h-40 overflow-y-auto p-4 rounded-lg border-2 border-white/20">
        <h3 class="text-center text-lg font-bold mb-3 text-white">🎤 Modo Karaoke 🎤</h3>
        <div class="space-y-2 text-center">
          <div 
            v-for="(linea, index) in letrasCancionActual" 
            :key="index"
            class="transition-all duration-300 text-lg font-medium"
            :class="index === lineaActualKaraoke ? 'text-yellow-300 text-xl scale-110 font-bold' : index < lineaActualKaraoke ? 'text-gray-400' : 'text-white/70'"
          >
            {{ linea.texto }}
          </div>
        </div>
      </div>

      <!-- CONTROL DE OPACIDAD DE VOZ (solo visible en modo karaoke) -->
      <div v-if="modoKaraoke" class="flex items-center space-x-3 w-full">
        <span class="text-sm text-white/80">🎵 Voz:</span>
        <input 
          type="range" 
          v-model="opacidadAudioOriginal" 
          @input="cambiarOpacidadAudio"
          min="0" 
          max="100"
          class="range range-xs flex-1 [--range-shdw:yellow] [--thumb-size:0.8rem]"
        />
        <span class="text-sm text-white/60 w-8 text-center">{{ opacidadAudioOriginal }}%</span>
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
      
      <!-- Control de Volumen -->
      <div class="flex items-center space-x-3 w-full">
        <button @click="toggleSilencio" class="btn btn-circle btn-sm border-none text-white/80 bg-gray-500/30 hover:bg-gray-500/50 transition-transform hover:scale-110">
          <i class="bi text-lg" :class="estaSilenciado || volumenActual == 0 ? 'bi-volume-mute-fill' : volumenActual < 50 ? 'bi-volume-down-fill' : 'bi-volume-up-fill'"></i>
        </button>
        
        <input 
          type="range" 
          v-model="volumenActual" 
          @input="cambiarVolumen"
          min="0" 
          max="100"
          class="range range-xs flex-1 [--range-shdw:white] [--thumb-size:0.8rem]"
        />
        
        <span class="text-sm text-white/60 w-8 text-center">{{ volumenActual }}</span>
      </div>
      
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

          <!-- BOTÓN DE KARAOKE -->
          <button @click="toggleKaraoke" class="btn btn-circle border-none text-white/80 transition-transform hover:scale-110" :class="modoKaraoke ? 'bg-yellow-500/60 text-yellow-100' : 'bg-gray-500/30 hover:bg-gray-500/50'">
            <i class="bi bi-mic-fill text-2xl"></i>
          </button>

          <!-- BOTÓN DE EDITOR -->
          <button @click="toggleEditor" class="btn btn-circle border-none text-white/80 transition-transform hover:scale-110" :class="modoEditor ? 'bg-blue-500/60 text-blue-100' : 'bg-gray-500/30 hover:bg-gray-500/50'">
            <i class="bi bi-pencil-fill text-2xl"></i>
          </button>
      </div>
    </section>
  </div>
</template>