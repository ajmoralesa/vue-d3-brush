<script setup>
import BrushSingle from "./components/brushSingle.vue";
import BrushMultiple from "./components/brushMultiple.vue";
</script>

<template>
  <div>
    <h1>Single brush</h1>
    <ul>
      <li>Press add random brush</li>
      <li>Zoom in / out using the mouse wheel</li>
      <li>The brush scales properly to zooming changes</li>
    </ul>
    <BrushSingle></BrushSingle>
  </div>

  <div>
    <h1>Multiple brushes</h1>
    <ul>
      <li>Start manually drawing brushes</li>
      <li>Zoom in / out using the mouse wheel</li>
      <li>The brushes are re-drawn overlapping :(</li>
    </ul>
    <BrushMultiple></BrushMultiple>
  </div>
</template>

<style scoped></style>
