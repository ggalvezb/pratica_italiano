<template>
  <v-container fluid>
    <v-col cols="12" md="4">
      <div class="text-h4 font-weight-bold my-4">Pratica Italiano</div>
      <div class="text-body-1 mb-4 text-medium-emphasis">
        Selecciona l’area di interesse e rispondi alle domande.
      </div>
    </v-col>
    
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
                Nueva pregunt
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
            Elige una temática a la izquierda para comenzar.
          </div>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const temas = [
  { title: 'Passato prossimo', value: 'pp', icon: 'mdi-clock-check-outline' },
  { title: 'Passato imperfetto', value: 'imp', icon: 'mdi-history' },
  { title: 'Condizionale', value: 'cond', icon: 'mdi-help-circle-outline' },
  { title: "C’è / Ce l’ho", value: 'celo', icon: 'mdi-comment-question-outline' },
  { title: 'Imperativo', value: 'impv', icon: 'mdi-gesture-tap' },
]

// Banco
const banco = {
  pp: [
    {
      texto: 'Ieri ______ una pizza deliziosa.',
      opciones: ['ho mangiato', 'mangiavo', 'mangerò', 'avrei mangiato', 'mangiai'],
      correcta: 'ho mangiato',
      explicacion: 'Si usa “ho mangiato” (passato prossimo) perché indica un’azione completata in un momento preciso del passato, in questo caso “ieri”.',
    },
    {
      texto: 'L’anno scorso ______ a Roma per lavoro.',
      opciones: ['vado', 'andavo', 'sono andato/a', 'andrei', 'andai'],
      correcta: 'sono andato/a',
      explicacion: 'Il passato prossimo (“sono andato/a”) si usa per un’azione conclusa nel passato. Con i verbi di movimento come “andare” si usa l’ausiliare “essere”.',
    },
    {
      texto: 'Abbiamo ______ il film ieri sera.',
      opciones: ['guardato', 'guardavamo', 'guarderemo', 'avremmo guardato', 'guardammo'],
      correcta: 'guardato',
      explicacion: '“Abbiamo guardato” è al passato prossimo e si usa per azioni finite e collocate nel tempo, come “ieri sera”.',
    },
  ],
  imp: [
    {
      texto: 'Da bambino ______ sempre al parco.',
      opciones: ['ho giocato', 'giocavo', 'giocherò', 'avrei giocato', 'giocai'],
      correcta: 'giocavo',
      explicacion: 'L’imperfetto (“giocavo”) si usa per azioni abituali o ripetute nel passato, come “da bambino”.',
    },
    {
      texto: 'Mentre lei ______, io leggevo.',
      opciones: ['cucinò', 'ha cucinato', 'cucinava', 'cucinerà', 'cucinerebbe'],
      correcta: 'cucinava',
      explicacion: 'L’imperfetto (“cucinava”) descrive un’azione in corso nel passato mentre ne avveniva un’altra (“io leggevo”).',
    },
    {
      texto: 'Quando ______ piccolo, avevo paura del buio.',
      opciones: ['sono', 'ero', 'sarò', 'sarei', 'fui'],
      correcta: 'ero',
      explicacion: '“Ero” è l’imperfetto di “essere” e si usa per descrivere stati o condizioni durature nel passato.',
    },
  ],
  cond: [
    {
      texto: 'Se avessi tempo, ______ più italiano.',
      opciones: ['studio', 'studiavo', 'ho studiato', 'studierò', 'studerei'],
      correcta: 'studerei',
      explicacion: 'Il condizionale presente (“studerei”) si usa per esprimere un’azione ipotetica che dipende da una condizione (“se avessi tempo”).',
    },
    {
      texto: 'Ti ______ volentieri, ma sono occupato.',
      opciones: ['aiuto', 'aiutavo', 'aiuterei', 'aiuterò', 'avrei aiutato'],
      correcta: 'aiuterei',
      explicacion: 'Il condizionale (“aiuterei”) serve per esprimere cortesia o disponibilità ipotetica: “ti aiuterei, ma sono occupato”.',
    },
    {
      texto: 'Al tuo posto, non ______ così tardi.',
      opciones: ['arrivo', 'arrivavo', 'arriverei', 'arriverò', 'arrivai'],
      correcta: 'arriverei',
      explicacion: 'Il condizionale (“arriverei”) si usa per dare consigli o indicare cosa si farebbe in una situazione ipotetica.',
    },
  ],
  celo: [
    {
      texto: '— Hai una penna? — Sì, ______.',
      opciones: ['ce la', "c’è lo", "ce l’ho", 'ce ho', 'celo'],
      correcta: "ce l’ho",
      explicacion: '“Ce l’ho” significa “ce l’ho” nel senso di “la possiedo”. È la forma corretta con il pronome combinato “ce + la/lo + ho”.',
    },
    {
      texto: '— Avete il libro? — Sì, ______ tutti.',
      opciones: ['ce li abbiamo', 'li abbiamo', 'ce le ho', 'c’è li', 'ci lo abbiamo'],
      correcta: 'ce li abbiamo',
      explicacion: '“Ce li abbiamo” unisce il pronome “li” (li oggetti) con “abbiamo”; “ce” si usa per motivi fonetici e di enfasi.',
    },
    {
      texto: '— Ha la mappa? — No, non ______.',
      opciones: ['ce l’ha', 'ce la', 'ce l’ho', 'ce l’ha più', 'ce l’abbiamo'],
      correcta: 'ce l’ha',
      explicacion: '“Ce l’ha” vuol dire “ce l’ha” nel senso di “la possiede”. È la fusione di “ce + la + ha”.',
    },
  ],
  impv: [
    {
      texto: '______ silenzio, per favore!',
      opciones: ['Fai', 'Fate', 'Faccia', 'Facciamo', 'Farò'],
      correcta: 'Fate',
      explicacion: '“Fate” è l’imperativo plurale di “fare”. Si usa per dare un ordine o una richiesta a più persone.',
    },
    {
      texto: '______ attenzione alla spiegazione!',
      opciones: ['Fare', 'Fate', 'Faccia', 'Fanno', 'Faranno'],
      correcta: 'Fate',
      explicacion: '“Fate attenzione” è un’espressione fissa per dire “state attenti”. È la forma imperativa plurale di “fare”.',
    },
    {
      texto: 'Non ______ tardi!',
      opciones: ['arrivare', 'arriva', 'arrivate', 'arrivi', 'arriverai'],
      correcta: 'arrivare',
      explicacion: 'Con il “tu” nell’imperativo negativo si usa l’infinito: “non arrivare” (non arrivare tardi).',
    },
  ],
}

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
</style>
