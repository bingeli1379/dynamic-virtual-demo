<script setup>
// base
import { computed } from 'vue'

// random fake height: 40 ~ 200
const itemMinHeight = parseFloat(import.meta.env.VITE_APP_ITEM_MIN_HEIGHT)
const randomHeight = Math.random() * 200
const height = randomHeight < itemMinHeight ? itemMinHeight : randomHeight

// style
const style = computed(() => ({
  height: `${height}px`,
  lineHeight: `${height}px`
}))
</script>

<template>
  <div :style="style">
    <slot />
  </div>
</template>
