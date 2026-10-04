<template>
  <main>
    <h1>Игровое поле</h1>
    <p>Карта {{ worldWidth }} × {{ worldHeight }} · шаг сетки {{ gridSize }}</p>

    <div class="field-viewport">
      <svg
          class="game-world"
          :width="projectedWidth"
          :height="projectedHeight"
          :viewBox="`${-projectedWidth / 2} ${-projectedHeight / 2} ${projectedWidth} ${projectedHeight}`"
          role="group"
          aria-labelledby="field-title field-description"
      >
        <title id="field-title">Координатная карта RTS</title>
        <desc id="field-description">
          Видна центральная часть изометрической карты. Начало координат в центре мира,
          X растёт вправо-вниз, Y — влево-вниз. Светлые линии обозначают оси,
          ромбовидная сетка отмечает шаг в {{ gridSize }} мировых единиц.
        </desc>

        <polygon
            :points="mapOutline"
            fill="#354c2e"
            stroke="#b9cba5"
            stroke-width="2"
        />
        <line
            v-for="(line, index) in gridLines"
            :key="index"
            :x1="line.start.x"
            :y1="line.start.y"
            :x2="line.end.x"
            :y2="line.end.y"
            :stroke="line.isAxis ? '#b9cba5' : '#66805b'"
            :stroke-width="line.isAxis ? 2 : 1"
        />
        <circle cx="0" cy="0" r="4" fill="#f0f4e7" />
        <g fill="#f0f4e7" font-size="14">
          <text x="10" y="-12">(0, 0)</text>
          <text :x="xLabel.x" :y="xLabel.y" dx="8" dy="-8">+X</text>
          <text :x="yLabel.x" :y="yLabel.y" dx="-8" dy="-8" text-anchor="end">+Y</text>
        </g>
      </svg>
    </div>

    <RouterLink :to="{ name: $routes.EXAMPLE }">Пример Vuex</RouterLink>
  </main>
</template>

<script setup lang="ts">
import {
  worldWidth,
  worldHeight,
  gridSize,
  worldToScreen,
  projectedWidth,
  projectedHeight,
  mapOutline,
  gridLines,
} from '@/game/map'

const xLabel = worldToScreen(160, 0)
const yLabel = worldToScreen(0, 160)
</script>

<style scoped>
.field-viewport {
  position: relative;
  height: clamp(320px, 70vh, 720px);
  margin: 16px 0;
  overflow: clip;
  border: 1px solid #66805b;
  background: #243421;
}

.game-world {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
</style>
