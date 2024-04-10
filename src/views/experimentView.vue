<template>
  <div id="experiment-Container">
    <InspectorContainer />
  </div>
</template>

<script setup lang="ts">
import InspectorContainer from "@/components/experiment.vue";
</script>

<style>
#inspector-container {
  display: grid;
  height: 100vh;
  gap: 20px;
  grid-template-rows: auto 1fr;
}
</style>
