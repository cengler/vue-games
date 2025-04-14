<template>
  <v-card class="robot-window">
    <v-card-title class="d-flex align-center">
      <v-icon start>mdi-robot</v-icon>
      Robot App
      <v-spacer></v-spacer>
      <v-btn icon @click="$emit('close')">
        <v-icon>mdi-close</v-icon>
      </v-btn>
    </v-card-title>
    <v-card-text class="pa-0">
      <RobotApp />
    </v-card-text>
  </v-card>
</template>

<script setup>
import RobotApp from './RobotApp.vue'
defineEmits(['close'])
</script>

<style scoped>
.robot-window {
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