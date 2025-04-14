<script setup>
import { ref } from 'vue'
import RobotWindow from './components/RobotWindow.vue'
import MinesweeperWindow from './components/MinesweeperWindow.vue'

const showRobotWindow = ref(false)
const showMinesweeperWindow = ref(false)

const desktopIcons = ref([
  {
    name: 'My Computer',
    icon: 'mdi-desktop-classic',
    action: () => console.log('My Computer clicked')
  },
  {
    name: 'Documents',
    icon: 'mdi-folder',
    action: () => console.log('Documents clicked')
  },
  {
    name: 'Recycle Bin',
    icon: 'mdi-delete',
    action: () => console.log('Recycle Bin clicked')
  },
  {
    name: 'Settings',
    icon: 'mdi-cog',
    action: () => console.log('Settings clicked')
  },
  {
    name: 'Robot',
    icon: 'mdi-robot',
    action: () => showRobotWindow.value = true
  },
  {
    name: 'Buscaminas',
    icon: 'mdi-bomb',
    action: () => showMinesweeperWindow.value = true
  }
])
</script>

<template>
  <v-app class="desktop-app">
    <v-main class="desktop-main pa-0">
      <div class="desktop-icons">
        <v-card
          v-for="icon in desktopIcons"
          :key="icon.name"
          class="desktop-icon"
          @click="icon.action"
        >
          <v-icon size="48" class="icon-image">{{ icon.icon }}</v-icon>
          <div class="icon-label">{{ icon.name }}</div>
        </v-card>
      </div>
    </v-main>

    <RobotWindow
      v-if="showRobotWindow"
      @close="showRobotWindow = false"
    />

    <MinesweeperWindow
      v-if="showMinesweeperWindow"
      @close="showMinesweeperWindow = false"
    />
  </v-app>
</template>

<style scoped>
.desktop-app {
  background-image: url('https://images.unsplash.com/photo-1557683316-973673baf926?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=1920&q=80');
  background-size: cover;
  background-position: center;
  height: 100vh;
  width: 100vw;
  margin: 0;
  padding: 0;
  overflow: hidden;
}

:deep(.v-application) {
  height: 100vh !important;
  width: 100vw !important;
  margin: 0 !important;
  padding: 0 !important;
}

:deep(.v-main) {
  padding: 0 !important;
  margin: 0 !important;
  height: 100vh !important;
  width: 100vw !important;
}

.desktop-main {
  height: 100vh;
  width: 100vw;
  overflow-y: auto;
  padding: 40px;
}

.desktop-icons {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
  gap: 30px;
  width: 100%;
  max-width: 1200px;
  margin: 30px;
}

.desktop-icon {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 15px;
  cursor: pointer;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(5px);
  border-radius: 8px;
  transition: all 0.3s ease;
  width: 100%;
  max-width: 120px;
  margin: 0 auto;
}

.desktop-icon:hover {
  background: rgba(255, 255, 255, 0.2);
  transform: scale(1.05);
}

.icon-image {
  margin-bottom: 8px;
}

.icon-label {
  color: white;
  text-align: center;
  font-size: 12px;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.5);
}
</style>
