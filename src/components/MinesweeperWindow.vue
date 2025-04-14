<template>
  <v-card class="minesweeper-window">
    <v-card-title class="d-flex align-center">
      <v-icon start>mdi-bomb</v-icon>
      Buscaminas
      <v-spacer></v-spacer>
      <v-btn icon @click="$emit('close')">
        <v-icon>mdi-close</v-icon>
      </v-btn>
    </v-card-title>
    <v-card-text class="pa-0">
      <MinesweeperApp />
    </v-card-text>
  </v-card>
</template>

<script setup>
import MinesweeperApp from './MinesweeperApp.vue'
defineEmits(['close'])
</script>

<style scoped>
.minesweeper-window {
  width: 80%;
  max-width: 90vw;
  height: 80%;
  max-height: 90vh;
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 1000;
}

:deep(.v-card-text) {
  height: calc(100% - 64px);
  overflow: hidden;
}
</style> 