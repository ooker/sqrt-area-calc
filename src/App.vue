<script setup>
import RectangleCalculator from './components/RectangleCalculator.vue'
</script>

<template>
  <main class="min-h-screen flex items-center justify-center p-4">
    <div class="max-w-4xl w-full">
      <h1 class="text-4xl font-extrabold text-center mb-2 text-transparent bg-clip-text bg-gradient-to-r from-teal-400 to-blue-500">
        Rectangle Area Calculator
      </h1>
      <p class="text-slate-400 text-center mb-8">Maintain a constant area while dynamically adjusting the sides.</p>
      
      <RectangleCalculator />
    </div>
  </main>
</template>
