<template>
  <v-container class="py-8" style="max-width: 1100px">
    <v-row>
      <!-- INPUTS -->
      <v-col cols="12" md="6">
        <v-card class="pa-6" elevation="6">
          <div class="text-h5 font-weight-bold">Pratica Italiano</div>
          <div class="text-body-2 mb-4 text-medium-emphasis">Seleziona l’area di interesse.</div>
          <v-divider class="my-4" />
          <!-- selecciona area de ejercicios -->
          <v-list
            v-model="selectedArea"
            :items="areas"
            label="Area"
            outlined
            dense
            class="mb-4"
          ></v-list>


        </v-card>
      </v-col>

      <!-- RESULTS -->
      <v-col cols="12" md="6">
        <v-row>
          <!-- Titulo de la pregunta -->
           <v-col cols="12">
            <div class="text-h5 font-weight-bold mb-4">Pregunta</div>
          </v-col>
        </v-row>
      </v-col>
    </v-row>


  </v-container>
</template>

<script setup>
import { reactive, computed, onMounted } from 'vue'
const areas = [
  'Passato Prossimo',
  'Passato Remoto',
  'Imperativo',
  'Condizionale',
  'C\'é lo',
]
</script>

<style scoped>

</style>
