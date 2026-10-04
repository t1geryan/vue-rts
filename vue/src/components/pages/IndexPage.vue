<template>
  <main>
    <h1>Игровое поле</h1>
    <p>Карта {{ worldWidth }} × {{ worldHeight }} · шаг сетки {{ gridSize }}</p>
    <p>Камера: удерживайте WASD или стрелки.</p>
    <p>ЛКМ по персонажу — выбрать, по пустому месту — снять выбор. ПКМ — идти к точке.</p>

    <div class="field-viewport" @click="unit.selected = false" @contextmenu.prevent>
      <svg
          class="game-world"
          :width="projectedWidth"
          :height="projectedHeight"
          :viewBox="`${cameraX - projectedWidth / 2} ${cameraY - projectedHeight / 2} ${projectedWidth} ${projectedHeight}`"
          role="group"
          aria-labelledby="field-title field-description"
          @contextmenu.prevent="moveUnit"
      >
        <title id="field-title">Координатная карта RTS</title>
        <desc id="field-description">
          Изометрическая карта с управлением камерой через WASD или стрелки. Начало координат в центре мира,
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
        <ellipse
            v-if="unit.x !== unit.targetX || unit.y !== unit.targetY"
            :cx="targetScreen.x"
            :cy="targetScreen.y"
            rx="7"
            ry="4"
            fill="none"
            stroke="#ffe08a"
            stroke-width="2"
            pointer-events="none"
        />
        <g
            class="unit"
            :transform="`translate(${unitScreen.x} ${unitScreen.y})`"
            role="button"
            aria-label="Персонаж"
            :aria-pressed="unit.selected"
            @click.stop="unit.selected = true"
        >
          <ellipse v-if="unit.selected" rx="18" ry="9" fill="none" stroke="#ffe08a" stroke-width="2" />
          <circle cy="-10" r="10" fill="#ffd376" stroke="#493618" stroke-width="2" />
        </g>
      </svg>
    </div>

    <RouterLink :to="{ name: $routes.EXAMPLE }">Пример Vuex</RouterLink>
  </main>
</template>

<script setup lang="ts">
import { computed, onMounted, onUnmounted, reactive, ref } from 'vue'
import { Unit } from '@/game/Unit'
import {
  worldWidth,
  worldHeight,
  gridSize,
  worldToScreen,
  screenToWorld,
  projectedWidth,
  projectedHeight,
  mapOutline,
  gridLines,
} from '@/game/map'

const xLabel = worldToScreen(160, 0)
const yLabel = worldToScreen(0, 160)

const unit = reactive(new Unit(80, 80, 150))
const unitScreen = computed(() => worldToScreen(unit.x, unit.y))
const targetScreen = computed(() => worldToScreen(unit.targetX, unit.targetY))

function moveUnit(event: MouseEvent) {
  if (!unit.selected) return
  const svg = event.currentTarget as SVGSVGElement
  const matrix = svg.getScreenCTM()
  if (!matrix) return

  // Обратная матрица SVG учитывает положение на странице и камеру в viewBox.
  const point = new DOMPoint(event.clientX, event.clientY).matrixTransform(matrix.inverse())
  const target = screenToWorld(point.x, point.y)
  unit.moveTo(
      Math.max(-worldWidth / 2, Math.min(worldWidth / 2, target.x)),
      Math.max(-worldHeight / 2, Math.min(worldHeight / 2, target.y)),
  )
}

// Камера хранит координаты центра видимой области после проекции.
const cameraX = ref(0)
const cameraY = ref(0)
const CAMERA_SPEED = 600 // экранных пикселей в секунду
const pressedKeys = new Set<string>()
const cameraKeys = ['KeyW', 'KeyA', 'KeyS', 'KeyD', 'ArrowUp', 'ArrowLeft', 'ArrowDown', 'ArrowRight']
let animationFrame = 0
let previousTime = 0

function onKeyDown(event: KeyboardEvent) {
  if (!cameraKeys.includes(event.code) || event.ctrlKey || event.metaKey || event.altKey) return
  if (event.target instanceof HTMLElement && event.target.closest('input, textarea, select, [contenteditable]')) return

  event.preventDefault()
  pressedKeys.add(event.code)
}

function onKeyUp(event: KeyboardEvent) {
  pressedKeys.delete(event.code)
}

function clearKeys() {
  pressedKeys.clear()
}

function updateFrame(time: number) {
  // Ограничение времени шага предотвращает скачок после неактивной вкладки.
  const delta = Math.max(0, Math.min((time - previousTime) / 1000, 0.05))
  previousTime = time
  const dx = Number(pressedKeys.has('KeyD') || pressedKeys.has('ArrowRight'))
      - Number(pressedKeys.has('KeyA') || pressedKeys.has('ArrowLeft'))
  const dy = Number(pressedKeys.has('KeyS') || pressedKeys.has('ArrowDown'))
      - Number(pressedKeys.has('KeyW') || pressedKeys.has('ArrowUp'))
  const distance = CAMERA_SPEED * delta / (Math.hypot(dx, dy) || 1)

  // Простые прямоугольные границы по размерам проекции карты.
  cameraX.value = Math.max(-projectedWidth / 2, Math.min(projectedWidth / 2, cameraX.value + dx * distance))
  cameraY.value = Math.max(-projectedHeight / 2, Math.min(projectedHeight / 2, cameraY.value + dy * distance))
  unit.update(delta)
  animationFrame = requestAnimationFrame(updateFrame)
}

onMounted(() => {
  window.addEventListener('keydown', onKeyDown)
  window.addEventListener('keyup', onKeyUp)
  window.addEventListener('blur', clearKeys)
  previousTime = performance.now()
  animationFrame = requestAnimationFrame(updateFrame)
})

onUnmounted(() => {
  window.removeEventListener('keydown', onKeyDown)
  window.removeEventListener('keyup', onKeyUp)
  window.removeEventListener('blur', clearKeys)
  cancelAnimationFrame(animationFrame)
  clearKeys()
})
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

.unit {
  cursor: pointer;
}
</style>
