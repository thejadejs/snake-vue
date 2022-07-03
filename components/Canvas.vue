<template>
  <div class="canvas">
    <div>
      <slot />
    </div>
  </div>
</template>

<style scoped lang="scss">
.canvas {
  width: 502px;
  height: 502px;
  background: pink;
  margin: 24px auto;
  & div {
    display: grid;
    grid-template: repeat(12, minmax(40px, 1fr)) / repeat(12, minmax(40px, 1fr));
    gap: 2px;
  }
}
</style>
