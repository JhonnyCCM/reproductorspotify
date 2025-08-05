<script setup>
import { ref, onMounted, computed, watch } from 'vue';

// Importación de los componentes
import SelectorArchivos from './components/SelectorArchivos.vue';
import Playlist from './components/Playlist.vue';
import VisorKaraoke from './components/VisorKaraoke.vue';
import EditorLetras from './components/EditorLetras.vue';
import InformacionCancion from './components/InformacionCancion.vue';
import ControlVolumen from './components/ControlVolumen.vue';
import ControlesPrincipales from './components/ControlesPrincipales.vue';

// --- ESTADO REACTIVO ---
const canciones = ref([]);
const indiceCancionActual = ref(0);
const modoAleatorio = ref(false);
const modoRepetir = ref(false);
const estaReproduciendo = ref(false);
const progresoActual = ref(0);
const volumenActual = ref(50);
const estaSilenciado = ref(false);
const volumenAnterior = ref(50);
const modoKaraoke = ref(false);
const lineaActualKaraoke = ref(0);
const opacidadAudioOriginal = ref(100);
const modoEditor = ref(false);
const letrasEnEdicion = ref([]);
const tiempoMarcado = ref(0);
const audioPlayer = ref(null);
const letrasKaraoke = ref({
  "default": [
    { tiempo: 0, texto: "♪ Bienvenido al modo karaoke ♪" },
    { tiempo: 3, texto: "Carga una canción para ver las letras" },
  ]
});

// --- DATOS COMPUTADOS ---
const cancionActual = computed(() => {
  return canciones.value[indiceCancionActual.value] || { titulo: 'Sin canción', nombre: 'Agrega música', fuente: '' };
});

const letrasCancionActual = computed(() => {
  return (cancionActual.value && letrasKaraoke.value[cancionActual.value.titulo]) || letrasKaraoke.value.default;
});

// --- WATCHER PARA CONTROL DE VOLUMEN ---
watch([volumenActual, opacidadAudioOriginal, modoKaraoke], () => {
  if (!audioPlayer.value) return;
  let volumenFinal = modoKaraoke.value
    ? (volumenActual.value * opacidadAudioOriginal.value) / 100
    : volumenActual.value;
  audioPlayer.value.volume = volumenFinal / 100;
  estaSilenciado.value = volumenActual.value === 0;
}, { immediate: true });


// --- MÉTODOS ---
function manejarCambioDeFondo(e) {
  const file = e.target.files[0];
  if (file) {
    const reader = new FileReader();
    reader.onload = (event) => { document.getElementById('app-container').style.backgroundImage = `url(${event.target.result})`; };
    reader.readAsDataURL(file);
  }
}

function manejarSeleccionDeMusica(e) {
  const files = Array.from(e.target.files);
  const nuevasCanciones = files.map(file => {
    const url = URL.createObjectURL(file);
    const nombreArchivo = file.name.replace(/\.[^/.]+$/, "");
    if (!letrasKaraoke.value[nombreArchivo]) {
      letrasKaraoke.value[nombreArchivo] = [{ tiempo: 0, texto: `Letras para ${nombreArchivo}` }];
    }
    return { titulo: nombreArchivo, nombre: 'Archivo local', fuente: url };
  });
  canciones.value.push(...nuevasCanciones);
  if (canciones.value.length === nuevasCanciones.length && nuevasCanciones.length > 0) {
    cargarCancion();
  }
}

// LÓGICA DE CARGA DE CANCIÓN CORREGIDA
function cargarCancion() {
  if (cancionActual.value && audioPlayer.value) {
    // Simplemente carga la nueva fuente. El listener 'loadeddata' se encargará del resto.
    audioPlayer.value.src = cancionActual.value.fuente;
    progresoActual.value = 0;
  }
}

function reproducirPausar() {
  if (!canciones.value.length || !audioPlayer.value.src) return;
  if (audioPlayer.value.paused) {
    reproducirCancion();
  } else {
    pausarCancion();
  }
}

function reproducirCancion() {
  if (!audioPlayer.value.src) return;
  audioPlayer.value.play()
    .then(() => { estaReproduciendo.value = true; })
    .catch(e => console.error("Error al reproducir:", e));
}

function pausarCancion() {
  audioPlayer.value.pause();
  estaReproduciendo.value = false;
}

function actualizarProgreso() {
  if (isNaN(audioPlayer.value.duration)) return;
  progresoActual.value = audioPlayer.value.currentTime;
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
  if (modoAleatorio.value) modoRepetir.value = false;
  if (audioPlayer.value) audioPlayer.value.loop = modoRepetir.value;
}

function toggleRepetir() {
  modoRepetir.value = !modoRepetir.value;
  if (modoRepetir.value) modoAleatorio.value = false;
  if (audioPlayer.value) audioPlayer.value.loop = modoRepetir.value;
}

function toggleSilencio() {
  if (estaSilenciado.value || volumenActual.value === 0) {
    volumenActual.value = volumenAnterior.value > 0 ? volumenAnterior.value : 50;
  } else {
    volumenAnterior.value = volumenActual.value;
    volumenActual.value = 0;
  }
}

function toggleKaraoke() {
  modoKaraoke.value = !modoKaraoke.value;
  if (modoKaraoke.value) {
    modoEditor.value = false;
    lineaActualKaraoke.value = 0;
  }
}

function actualizarLineaKaraoke() {
  const tiempoActual = audioPlayer.value.currentTime;
  const letras = letrasCancionActual.value;
  for (let i = letras.length - 1; i >= 0; i--) {
    if (tiempoActual >= letras[i].tiempo) {
      if (lineaActualKaraoke.value !== i) lineaActualKaraoke.value = i;
      break;
    }
  }
}

function toggleEditor() {
  modoEditor.value = !modoEditor.value;
  if (modoEditor.value) {
    modoKaraoke.value = false;
    cargarLetrasExistentes();
  }
}

function cargarLetrasExistentes() {
  letrasEnEdicion.value = JSON.parse(JSON.stringify(letrasCancionActual.value.filter(l => !l.texto.includes('♪'))));
}

function marcarTiempo() {
  if (!audioPlayer.value) return;
  tiempoMarcado.value = parseFloat(audioPlayer.value.currentTime.toFixed(1));
}

function agregarLinea(texto) {
  if (!texto.trim()) return;
  const nuevaLinea = { tiempo: tiempoMarcado.value, texto: texto.trim() };
  letrasEnEdicion.value.push(nuevaLinea);
  letrasEnEdicion.value.sort((a, b) => a.tiempo - b.tiempo);
}

function eliminarLinea(index) {
  letrasEnEdicion.value.splice(index, 1);
}

function guardarLetras() {
  if (!cancionActual.value) return;
  letrasKaraoke.value[cancionActual.value.titulo] = [...letrasEnEdicion.value];
  alert('¡Letras guardadas!');
}

function limpiarEditor() {
  letrasEnEdicion.value = [];
  textoLineaActual.value = '';
  tiempoMarcado.value = 0;
}

function formatearTiempo(segundos) {
  const min = Math.floor(segundos / 60);
  const seg = Math.floor(segundos % 60);
  return `${String(min).padStart(2, '0')}:${String(seg).padStart(2, '0')}`;
}

function manejarFinDeCancion() {
  if (!audioPlayer.value.loop) {
    siguienteCancion();
  }
}

onMounted(() => {
  if (audioPlayer.value) {
    // LISTENER DE DATOS CARGADOS (CLAVE PARA REPRODUCCIÓN AUTOMÁTICA)
    audioPlayer.value.addEventListener('loadeddata', () => {
      const elAudio = audioPlayer.value;
      if (elAudio && !isNaN(elAudio.duration)) {
        progresoActual.max = elAudio.duration;
        // Si el estado global indica que debe sonar, se reproduce.
        if (estaReproduciendo.value) {
          reproducirCancion();
        }
      }
    });
    audioPlayer.value.addEventListener('ended', manejarFinDeCancion);
  }
});
</script>

<template>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.0/font/bootstrap-icons.css">

  <div id="app-container" class="flex justify-center items-center min-h-screen w-full bg-cover bg-center" style="background-image: url(/BackGround.jpg);">
    <div class="absolute inset-0 bg-black/50 backdrop-blur-md z-10"></div>
    
    <section class="z-20 flex flex-col text-gray-200 w-[380px] p-6 rounded-2xl bg-white/10">
      
      <div v-if="!modoKaraoke && !modoEditor" class="contents">
        <div class="flex flex-col w-full space-y-4">
          <SelectorArchivos
            @seleccionar-musica="manejarSeleccionDeMusica" 
            @seleccionar-fondo="manejarCambioDeFondo" 
          />
          <Playlist
            :canciones="canciones" 
            :indice-cancion-actual="indiceCancionActual"
            @seleccionar-cancion="tocarDesdePlaylist"
          />
          <InformacionCancion :cancion="cancionActual" />
          <input 
            type="range" 
            v-model="progresoActual" 
            @input="setProgreso"
            :max="audioPlayer && !isNaN(audioPlayer.duration) ? audioPlayer.duration : 100"
            step="0.1"
            class="range range-xs [--range-shdw:white] [--thumb-size:1rem]"
          />
          <ControlVolumen 
            v-model:volumen="volumenActual"
            :esta-silenciado="estaSilenciado"
            @toggle-silencio="toggleSilencio"
          />
        </div>
      </div>
      
      <div v-if="modoEditor" class="contents">
        <EditorLetras
          :letras-en-edicion="letrasEnEdicion"
          :tiempo-marcado="tiempoMarcado"
          :formatear-tiempo="formatearTiempo"
          @marcar-tiempo="marcarTiempo"
          @agregar-linea="agregarLinea"
          @eliminar-linea="eliminarLinea"
          @guardar="guardarLetras"
          @limpiar="limpiarEditor"
          @cerrar="toggleEditor"
        />
      </div>

      <div v-if="modoKaraoke" class="contents">
        <VisorKaraoke
          :letras="letrasCancionActual"
          :linea-actual="lineaActualKaraoke"
          v-model:opacidad-voz="opacidadAudioOriginal"
        />
      </div>

      <div v-if="!modoEditor" class="mt-4 w-full">
         <ControlesPrincipales
          :esta-reproduciendo="estaReproduciendo"
          :modo-aleatorio="modoAleatorio"
          :modo-repetir="modoRepetir"
          :modo-karaoke="modoKaraoke"
          :modo-editor="modoEditor"
          @reproducir-pausar="reproducirPausar"
          @anterior="cancionAnterior"
          @siguiente="siguienteCancion"
          @toggle-aleatorio="toggleAleatorio"
          @toggle-repetir="toggleRepetir"
          @toggle-karaoke="toggleKaraoke"
          @toggle-editor="toggleEditor"
        />
      </div>

      <audio 
        ref="audioPlayer" 
        @timeupdate="actualizarProgreso"
        class="hidden"
      ></audio>

    </section>
  </div>
</template>