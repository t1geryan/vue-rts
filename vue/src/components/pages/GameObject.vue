<template>
  <g
      class="building"
      role="button"
      aria-label="Здание"
      :aria-pressed="selected"
      tabindex="0"
      @click.stop="emit('select')"
      @contextmenu.stop.prevent="emit('target', $event.shiftKey)"
      @keydown.enter.stop.prevent="emit('select')"
      @keydown.space.stop.prevent="emit('select')"
  >
    <polygon
        :class="['outline', { 'outline--selected': selected }]"
        :points="points"
        :fill="groupColor(object.group)"
    />
    <text class="health" :x="top.x" :y="top.y - 8" text-anchor="middle">
      {{ object.health }}/{{ object.maxHealth }} HP
    </text>
  </g>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { objectCorners, type MapObject } from '@/game/occupancy'
import { groupColor } from '@/game/groups'

const props = defineProps<{ object: MapObject; selected?: boolean }>()
const emit = defineEmits<{ select: []; target: [queue: boolean] }>()
const corners = computed(() => objectCorners(props.object))
const points = computed(() => corners.value.map(point => `${point.x},${point.y}`).join(' '))
const top = computed(() => corners.value[0])
</script>

<style scoped lang="scss">
.building {
  cursor: pointer;

  &:focus:not(:focus-visible) {
    outline: none;
  }

  .outline {
    fill-opacity: 0.6;
    stroke: $color-outline-dark;
    stroke-width: 2;

    &--selected {
      stroke: $color-highlight;
    }
  }

  .health {
    fill: $color-label;
    font-size: 11px;
    pointer-events: none;
  }
}
</style>
