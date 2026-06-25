<script setup lang="ts">
defineProps<{
  title: string
  description: string
  tech: string[]
  status: string
  statusColor?: string
  icon?: string
}>()
</script>

<template>
  <div class="glass-card flex flex-col gap-3">
    <div class="flex items-center justify-between">
      <h3 class="text-xl font-bold m-0">{{ title }}</h3>
      <span
        class="tag"
        :style="{ color: statusColor || '#0070f3', borderColor: statusColor || '#0070f3' }"
      >
        {{ status }}
      </span>
    </div>
    <p class="text-sm opacity-70 m-0 leading-relaxed">{{ description }}</p>
    <div class="flex flex-wrap gap-2 mt-auto">
      <span
        v-for="t in tech"
        :key="t"
        class="tag text-xs opacity-60"
      >
        {{ t }}
      </span>
    </div>
  </div>
</template>
