<script setup lang="ts">
defineProps<{
  value: string | number
  label: string
  color?: string
}>()
</script>

<template>
  <div class="flex flex-col items-center gap-1 px-6 py-4 rounded-xl" style="background: rgba(255,255,255,0.04)">
    <span class="text-3xl font-bold" :style="{ color: color || '#0070f3' }">{{ value }}</span>
    <span class="text-xs uppercase tracking-wider opacity-40">{{ label }}</span>
  </div>
</template>
