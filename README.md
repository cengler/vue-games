# Robot Game

Un juego educativo donde programamos un robot para recolectar frutillas en una grilla con obstáculos.

## Descripción

Este juego permite a los usuarios programar un robot mediante instrucciones simples para navegar por una grilla y recolectar frutillas. El juego incluye:

- Una grilla de 5x10 celdas
- Un robot que puede moverse en cuatro direcciones (Norte, Sur, Este, Oeste)
- Frutillas que aparecen aleatoriamente
- Paredes/obstáculos que el robot debe evitar
- Sistema de programación con instrucciones básicas
- Efectos de sonido para cada movimiento

## Instrucciones de Juego

1. **Programación del Robot**:
   - Usa los botones para agregar instrucciones:
     - `IZQUIERDA`: Gira el robot 90° a la izquierda
     - `DERECHA`: Gira el robot 90° a la derecha
     - `ADELANTE`: Mueve el robot una celda hacia adelante
   - Puedes agregar números para repetir la última instrucción
   - Las instrucciones se ejecutan en orden

2. **Objetivo**:
   - Programar el robot para que recolecte la frutilla
   - Evitar chocar con las paredes
   - Cada frutilla recolectada suma un punto

3. **Controles**:
   - `Iniciar`: Comienza la ejecución del programa
   - `Reiniciar`: Vuelve a empezar el juego
   - `Borrar todas`: Elimina todas las instrucciones

## Requisitos Técnicos

- Node.js (versión 14 o superior)
- npm o yarn

## Instalación

1. Clona el repositorio:
```bash
git clone [URL_DEL_REPOSITORIO]
cd vue-games
```

2. Instala las dependencias:
```bash
npm install
# o
yarn install
```

3. Inicia el servidor de desarrollo:
```bash
npm run dev
# o
yarn dev
```

4. Abre tu navegador en `http://localhost:5173`

## Tecnologías Utilizadas

- Vue 3
- Vuetify 3
- Vite

## Autor

Christian Engler

## Licencia

Este proyecto está bajo la Licencia MIT.
