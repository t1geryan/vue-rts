<template>
  <g v-if="unit.selected && queuePoints.length" class="queue">
    <polyline class="queue__line" :points="queueLine" />
    <g v-for="(point, index) in queuePoints" :key="index" :transform="`translate(${point.x} ${point.y})`">
      <ellipse class="queue__marker" rx="9" ry="5" />
      <text class="queue__number" y="3.5" text-anchor="middle">{{ index + 1 }}</text>
    </g>
  </g>
  <ellipse
      v-if="moveEvent"
      class="target"
      :cx="targetScreen.x"
      :cy="targetScreen.y"
      rx="7"
      ry="4"
  />
  <g
      class="unit"
      :transform="`translate(${unitScreen.x} ${unitScreen.y})`"
      role="button"
      aria-label="Персонаж"
      :aria-pressed="unit.selected"
      tabindex="0"
      @click.stop="emit('select', $event.shiftKey)"
      @contextmenu.stop.prevent="emit('target', $event.shiftKey)"
      @keydown.enter.stop.prevent="emit('select', $event.shiftKey)"
      @keydown.space.stop.prevent="emit('select', $event.shiftKey)"
  >
    <ellipse v-if="unit.selected" class="selection" rx="18" ry="9" />
    <circle
        :class="['body', { 'body--water': unit.terrain === 'water' }]"
        cy="-10"
        :r="UNIT_RADIUS - 1"
        :fill="groupColor(unit.group)"
    />
    <g v-if="unit.health < unit.maxHealth" class="health">
      <rect class="health__track" x="-12" y="-29" width="24" height="4" />
      <rect class="health__value" x="-12" y="-29" :width="24 * unit.health / unit.maxHealth" height="4" />
    </g>
  </g>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import type { Unit, MoveEvent } from '@/game/Unit'
import { UNIT_RADIUS } from '@/game/occupancy'
import { worldToScreen } from '@/game/map'
import { groupColor } from '@/game/groups'

const props = defineProps<{ unit: Unit; moveEvent?: MoveEvent; queue?: MoveEvent[] }>()
const emit = defineEmits<{ select: [additive: boolean]; target: [queue: boolean] }>()

const unitScreen = computed(() => worldToScreen(props.unit.x, props.unit.y))
const targetScreen = computed(() => worldToScreen(
  props.moveEvent?.x ?? props.unit.x,
  props.moveEvent?.y ?? props.unit.y,
))
const queuePoints = computed(() => (props.queue ?? []).map(move => worldToScreen(move.x, move.y)))
const queueLine = computed(() => [targetScreen.value, ...queuePoints.value]
  .map(point => `${point.x},${point.y}`).join(' '))
</script>

<style scoped lang="scss">
.queue {
  pointer-events: none;

  &__line {
    fill: none;
    stroke: $color-highlight;
    stroke-width: 1.5;
    stroke-dasharray: 6 4;
  }

  &__marker {
    fill: $color-ground-dark;
    stroke: $color-highlight;
    stroke-width: 1.5;
  }

  &__number {
    fill: $color-highlight;
    font-size: 10px;
  }
}

.target {
  fill: none;
  stroke: $color-highlight;
  stroke-width: 2;
  pointer-events: none;
}

.unit {
  cursor: pointer;

  &:focus:not(:focus-visible) {
    outline: none;
  }

  .selection {
    fill: none;
    stroke: $color-highlight;
    stroke-width: 2;
  }

  .body {
    stroke: $color-unit-land-stroke;
    stroke-width: 2;

    &--water {
      stroke: $color-unit-water-stroke;
      stroke-dasharray: 4 2;
    }
  }

  .health {
    pointer-events: none;

    &__track {
      fill: $color-outline-dark;
    }

    &__value {
      fill: $color-danger;
    }
  }
}
</style>
