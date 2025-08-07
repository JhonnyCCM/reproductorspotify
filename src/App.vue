<script setup>
import { ref, onMounted, computed, watch } from 'vue';

import SelectorArchivos from './components/SelectorArchivos.vue';
import Playlist from './components/Playlist.vue';
import VisorKaraoke from './components/VisorKaraoke.vue';
import EditorLetras from './components/EditorLetras.vue';
import InformacionCancion from './components/InformacionCancion.vue';
import ControlVolumen from './components/ControlVolumen.vue';
import ControlesPrincipales from './components/ControlesPrincipales.vue';

// --- ESTADO REACTIVO ---
const canciones = ref([]);
const indiceCancionActual = ref(-1);
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
const generandoFondo = ref(false);

// --- IA: generar fondo y convertirlo en base64 ---
async function generarFondoConIA(prompt) {
  generandoFondo.value = true;
  const apiToken = import.meta.env.VITE_HF_TOKEN;
  const modelo = 'black-forest-labs/FLUX.1-schnell'; // ✅ modelo gratuito
  const url = `https://api-inference.huggingface.co/models/${modelo}`;

  try {
    const resp = await fetch(url, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${apiToken}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ inputs: prompt }),
    });

    if (!resp.ok) return null;

    const blob = await resp.blob();
    const reader = new FileReader();
    return await new Promise((resolve, reject) => {
      reader.onloadend = () => resolve(reader.result);
      reader.onerror = reject;
      reader.readAsDataURL(blob);
    });

  } catch (error) {
    return null;
  } finally {
    generandoFondo.value = false;
  }
}

// --- COMPUTEDS ---
const cancionActual = computed(() => canciones.value[indiceCancionActual.value] || null);
const letrasCancionActual = computed(() =>
  (cancionActual.value && letrasKaraoke.value[cancionActual.value.titulo]) || letrasKaraoke.value.default
);

// --- WATCHERS ---
watch([volumenActual, opacidadAudioOriginal, modoKaraoke], () => {
  if (!audioPlayer.value) return;
  let volumenFinal = modoKaraoke.value ? (volumenActual.value * opacidadAudioOriginal.value) / 100 : volumenActual.value;
  audioPlayer.value.volume = volumenFinal / 100;
  estaSilenciado.value = volumenActual.value === 0;
});

watch(indiceCancionActual, (nuevoIndice) => {
  if (nuevoIndice !== -1) {
    cargarCancion();
  }
});

// --- FUNCIONES ---
function manejarCambioDeFondo(e) {
  const file = e.target.files[0];
  if (file) {
    const reader = new FileReader();
    reader.onload = (event) => {
      const fondoUrl = event.target.result;
      document.getElementById('app-container').style.backgroundImage = `url(${fondoUrl})`;
      if (cancionActual.value) {
        cancionActual.value.fondoUrl = fondoUrl;
      }
    };
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
    return { titulo: nombreArchivo, nombre: 'Archivo local', fuente: url, fondoUrl: null };
  });
  canciones.value.push(...nuevasCanciones);
  if (indiceCancionActual.value === -1 && canciones.value.length > 0) {
    indiceCancionActual.value = 0;
  }
}

async function cargarCancion() {
  const cancion = cancionActual.value;
  const letrasGuardadas = JSON.parse(localStorage.getItem('letrasKaraoke') || '{}');
  if (letrasGuardadas[cancion.titulo]) {
    letrasKaraoke.value[cancion.titulo] = letrasGuardadas[cancion.titulo];
  }

  if (cancion && audioPlayer.value) {
    const estabaReproduciendo = estaReproduciendo.value;
    const fondosGuardados = JSON.parse(localStorage.getItem('fondosCanciones') || '{}');

    if (fondosGuardados[cancion.titulo]) {
      cancion.fondoUrl = fondosGuardados[cancion.titulo];
      document.getElementById('app-container').style.backgroundImage = `url(${cancion.fondoUrl})`;
    } else {
      document.getElementById('app-container').style.backgroundImage = `url(/BackGround.jpg)`;
      const prompt = `Fondo de pantalla cinemático para la canción '${cancion.titulo}', arte digital.`;
      const nuevaUrlFondo = await generarFondoConIA(prompt);

      if (nuevaUrlFondo) {
        document.getElementById('app-container').style.backgroundImage = `url(${nuevaUrlFondo})`;
        cancion.fondoUrl = nuevaUrlFondo;
        fondosGuardados[cancion.titulo] = nuevaUrlFondo;
        localStorage.setItem('fondosCanciones', JSON.stringify(fondosGuardados));
      }
    }

    audioPlayer.value.src = cancion.fuente;
    audioPlayer.value.load();
    if (estabaReproduciendo) reproducirCancion();
  }
}

function reproducirPausar() {
  if (!cancionActual.value || !audioPlayer.value.src) return;
  audioPlayer.value.paused ? reproducirCancion() : pausarCancion();
}
function reproducirCancion() {
  if (!audioPlayer.value.src) return;
  audioPlayer.value.play().then(() => { estaReproduciendo.value = true; }).catch(console.error);
}
function pausarCancion() {
  audioPlayer.value.pause();
  estaReproduciendo.value = false;
}
function actualizarProgreso() {
  if (isNaN(audioPlayer.value.duration)) return;
  progresoActual.value = audioPlayer.value.currentTime;
  if (modoKaraoke.value) actualizarLineaKaraoke();
}
function setProgreso() {
  audioPlayer.value.currentTime = progresoActual.value;
}
function siguienteCancion() {
  if (canciones.value.length <= 1) return;
  indiceCancionActual.value = modoAleatorio.value ? obtenerIndiceAleatorio() : (indiceCancionActual.value + 1) % canciones.value.length;
}
function cancionAnterior() {
  if (canciones.value.length <= 1) return;
  indiceCancionActual.value = modoAleatorio.value ? obtenerIndiceAleatorio() : (indiceCancionActual.value - 1 + canciones.value.length) % canciones.value.length;
}
function tocarDesdePlaylist(index) {
  indiceCancionActual.value = index;
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
}
function toggleRepetir() {
  modoRepetir.value = !modoRepetir.value;
  if (modoRepetir.value) modoAleatorio.value = false;
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
  tiempoMarcado.value = parseFloat(audioPlayer.value.currentTime.toFixed(1));
}
function agregarLinea(texto) {
  if (!texto.trim()) return;
  letrasEnEdicion.value.push({ tiempo: tiempoMarcado.value, texto: texto.trim() });
  letrasEnEdicion.value.sort((a, b) => a.tiempo - b.tiempo);
}
function eliminarLinea(index) {
  letrasEnEdicion.value.splice(index, 1);
}
function guardarLetras() {
  if (!cancionActual.value) return;

  const titulo = cancionActual.value.titulo;
  const letras = [...letrasEnEdicion.value];

  // Guarda en el estado reactivo
  letrasKaraoke.value[titulo] = letras;

  // Guarda en localStorage
  const letrasGuardadas = JSON.parse(localStorage.getItem('letrasKaraoke') || '{}');
  letrasGuardadas[titulo] = letras;
  localStorage.setItem('letrasKaraoke', JSON.stringify(letrasGuardadas));

  alert('¡Letras guardadas!');
}
function limpiarEditor() {
  letrasEnEdicion.value = [];
  tiempoMarcado.value = 0;
}
function formatearTiempo(segundos) {
  const min = Math.floor(segundos / 60);
  const seg = Math.floor(segundos % 60);
  return `${String(min).padStart(2, '0')}:${String(seg).padStart(2, '0')}`;
}
function manejarFinDeCancion() {
  if (!audioPlayer.value.loop) siguienteCancion();
}

onMounted(() => {
  const letrasGuardadas = JSON.parse(localStorage.getItem('letrasKaraoke') || '{}');
  letrasKaraoke.value = { ...letrasKaraoke.value, ...letrasGuardadas };

  if (audioPlayer.value) {
    audioPlayer.value.addEventListener('loadedmetadata', () => {
      progresoActual.value = 0;
      if (estaReproduciendo.value) reproducirCancion();
    });
    audioPlayer.value.addEventListener('ended', manejarFinDeCancion);
  }
});
</script>

<template>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.0/font/bootstrap-icons.css">

  <div id="app-container" class="flex justify-center items-center min-h-screen w-full bg-cover bg-center" style="background-image: url(/BackGround.jpg);">
    <div class="absolute inset-0 bg-black/50 backdrop-blur-md z-10"></div>

    <section class="z-20 flex flex-col items-center text-gray-200 w-[380px] p-6 rounded-2xl bg-white/10 space-y-4">
      <template v-if="!modoKaraoke && !modoEditor">
        <SelectorArchivos @seleccionar-musica="manejarSeleccionDeMusica" @seleccionar-fondo="manejarCambioDeFondo" />
        <Playlist :canciones="canciones" :indice-cancion-actual="indiceCancionActual" @seleccionar-cancion="tocarDesdePlaylist" />
        <InformacionCancion :cancion="cancionActual" />
        <input type="range" v-model="progresoActual" @input="setProgreso"
          :max="audioPlayer && !isNaN(audioPlayer.duration) ? audioPlayer.duration : 100"
          step="0.1" class="range range-xs [--range-shdw:white] [--thumb-size:1rem]" />
        <ControlVolumen v-model:volumen="volumenActual" :esta-silenciado="estaSilenciado" @toggle-silencio="toggleSilencio" />
      </template>

      <template v-if="modoEditor">
        <EditorLetras :letras-en-edicion="letrasEnEdicion" :tiempo-marcado="tiempoMarcado" :formatear-tiempo="formatearTiempo"
          @marcar-tiempo="marcarTiempo" @agregar-linea="agregarLinea" @eliminar-linea="eliminarLinea"
          @guardar="guardarLetras" @limpiar="limpiarEditor" @cerrar="toggleEditor" />
      </template>

      <template v-if="modoKaraoke">
        <VisorKaraoke :letras="letrasCancionActual" :linea-actual="lineaActualKaraoke" v-model:opacidad-voz="opacidadAudioOriginal" />
      </template>

      <div class="w-full pt-4">
        <ControlesPrincipales :esta-reproduciendo="estaReproduciendo" :modo-aleatorio="modoAleatorio"
          :modo-repetir="modoRepetir" :modo-karaoke="modoKaraoke" :modo-editor="modoEditor"
          @reproducir-pausar="reproducirPausar" @anterior="cancionAnterior" @siguiente="siguienteCancion"
          @toggle-aleatorio="toggleAleatorio" @toggle-repetir="toggleRepetir" @toggle-karaoke="toggleKaraoke"
          @toggle-editor="toggleEditor" />
      </div>

      <audio ref="audioPlayer" @timeupdate="actualizarProgreso" class="hidden"></audio>
    </section>
  </div>
</template>
