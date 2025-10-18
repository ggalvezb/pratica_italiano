<template>
  <v-toolbar flat color="#3A5A40" class="pa-0 mb-4">
    <v-toolbar-title class="text-h5">Pratica Italiano</v-toolbar-title>
  </v-toolbar>

  <v-container fluid>

    
    <v-row>
      <!-- Columna izquierda: Temáticas -->
      <v-col cols="12" md="4">
        <v-card class="pa-4">
          <h2 class="text-h6 mb-2">Seleziona l’area di interesse</h2>

          <v-list nav>
            <v-list-item
              v-for="t in temas"
              :key="t.value"
              :title="t.title"
              :prepend-icon="t.icon"
              :class="{
                'bg-green-lighten-4': selectedTema === t.value,
                'rounded-lg': true,
              }"
              @click="seleccionarTema(t.value)"
            />
          </v-list>
        </v-card>
      </v-col>

      <!-- Columna derecha: Pregunta + alternativas -->
      <v-col cols="12" md="8">
        <v-card class="pa-6" min-height="420">
          <div v-if="preguntaActual">
            <!-- MÓVIL: botón arriba -->
            <div class="d-flex justify-end mb-2 d-md-none">
              <v-btn
                variant="elevated"
                @click="nuevaPregunta"
                :disabled="!selectedTema"
              >
                Nueva pregunta
              </v-btn>
            </div>

            <!-- ESCRITORIO: título + botón al lado -->
            <div class="d-none d-md-flex justify-space-between align-center mb-4">
              <h2 class="text-h5 m-0">{{ preguntaActual.texto }}</h2>
              <v-btn
                variant="elevated"
                @click="nuevaPregunta"
                :disabled="!selectedTema"
              >
                Nuova domanda
              </v-btn>
            </div>

            <!-- MÓVIL: título debajo del botón -->
            <div class="d-md-none mb-4">
              <h2 class="text-h5 text-center m-0">{{ preguntaActual.texto }}</h2>
            </div>


            <v-radio-group v-model="respuesta" @update:model-value="onElegir">
              <v-radio
                v-for="(opt, idx) in preguntaActual.opciones"
                :key="idx"
                :label="opt"
                :value="opt"
              />
            </v-radio-group>

            <v-alert
              v-if="feedback"
              :type="feedback.correcto ? 'success' : 'error'"
              class="mt-4"
              variant="tonal"
            >
              {{ feedback.mensaje }}
            </v-alert>
          </div>

          <div v-else class="text-medium-emphasis">
            Scegli un argomento a sinistra per iniziare..
          </div>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import { banco } from '@/data/banco_a2.js'

const temas = [
  { title: 'Passato prossimo', value: 'pp', icon: 'mdi-clock-check-outline' },
  { title: 'Passato imperfetto', value: 'imp', icon: 'mdi-history' },
  { title: 'Condizionale', value: 'cond', icon: 'mdi-help-circle-outline' },
  { title: "C’è / Ce l’ho", value: 'celo', icon: 'mdi-comment-question-outline' },
  { title: 'Imperativo', value: 'impv', icon: 'mdi-gesture-tap' },
]

// Banco

const selectedTema = ref(null)
const respuesta = ref(null)
const feedback = ref(null)
const preguntaActual = ref(null)

// Para evitar repetir inmediatamente la misma pregunta
const ultimoIndice = ref({ pp: null, imp: null, cond: null, celo: null, impv: null })

function randomIndexExcept(len, except) {
  if (len <= 1 || except === null || except === undefined) return Math.floor(Math.random() * len)
  let idx
  do { idx = Math.floor(Math.random() * len) } while (idx === except)
  return idx
}

function nuevaPregunta() {
  if (!selectedTema.value) return
  const lista = banco[selectedTema.value]
  const idx = randomIndexExcept(lista.length, ultimoIndice.value[selectedTema.value])
  ultimoIndice.value[selectedTema.value] = idx
  preguntaActual.value = lista[idx]
  respuesta.value = null
  feedback.value = null
}

function seleccionarTema(value) {
  selectedTema.value = selectedTema.value === value ? null : value
  if (selectedTema.value) {
    nuevaPregunta()
  } else {
    preguntaActual.value = null
    respuesta.value = null
    feedback.value = null
  }
}

function onElegir() {
  if (!preguntaActual.value) return
  const ok = respuesta.value === preguntaActual.value.correcta
  feedback.value = {
    correcto: ok,
    mensaje: ok ? '✅ ¡Correcto!' : `❌ Incorrecto. Respuesta: ${preguntaActual.value.correcta} (${preguntaActual.value.explicacion})`,
  }
}

// Por si quieres cambiar de tema desde fuera y reaccionar:
watch(selectedTema, (v) => {
  if (v) nuevaPregunta()
})
</script>

<style scoped>
.v-list-item--active,
.bg-green-lighten-4 {
  transition: background 0.2s ease;
}
</style>@/data/banco_a2.js
