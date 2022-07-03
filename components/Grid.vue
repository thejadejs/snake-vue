<template>
  <div
    :class="['grid', { food }]"
    :style="
      pos
        ? {
            gridArea: `${pos.y || 1} / ${pos.x || 1}`,
          }
        : {}
    "
  >
    <slot />
  </div>
</template>

<script setup>
defineProps({ food: Boolean, pos: {} });
</script>

<style scoped lang="scss">
.grid {
  width: 100%;
  height: 100%;
  min-width: 40px;
  min-height: 40px;
  background: yellow;
  font-size: 0.7rem;
  transition: all 0.2ms;
  &.food {
    background: skyblue;
  }
}
</style>
