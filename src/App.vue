<template>
  <div id="app">
    <ComputerStatusChecker />
  </div>
</template>

<script setup>
import ComputerStatusChecker from './components/ComputerStatusChecker.vue'
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

#app {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}

body {
  margin: 0;
  font-family: 'SF Pro Display', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: #000;
  overflow: hidden;
}
</style>
