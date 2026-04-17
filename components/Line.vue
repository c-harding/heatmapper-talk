<script setup lang="ts">
import { computed } from 'vue';

interface Layer {
  color: string;
  opacity: number;
}

const GRID_STEP = 20;

const {
  count = 1,
  width,
  layers,
  color,
  opacity,
} = defineProps<{
  count?: number;
  width?: number;
  layers?: Layer[];
  color?: string;
  opacity?: number;
}>();

const resolvedWidth = computed(() => width || count + 1);

const resolvedLayer = computed(() => {
  if (layers) {
    return layers;
  }

  return [
    {
      color: color || 'currentColor',
      opacity: opacity || 1,
    },
  ];
});

const totalWidth = computed(() => resolvedWidth.value * GRID_STEP);

function makePath(i: number) {
  const offset = i * GRID_STEP;

  return `M${offset},4 a 12,12 0 0 0 12,12 H ${totalWidth.value}`;
}

const viewBox = computed(() => `0 0 ${totalWidth.value} 20`);
</script>

<template>
  <svg :viewBox="viewBox">
    <template v-for="(layer, j) in resolvedLayer" :key="j">
      <path
        v-for="i of count"
        :key="i"
        fill="transparent"
        :stroke="layer.color"
        :stroke-opacity="layer.opacity"
        stroke-width="8"
        :d="makePath(i)"
      />
    </template>
  </svg>
</template>
