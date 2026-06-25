<script setup lang="ts">
defineProps<{
  items: { title: string; description: string }[]
}>()
</script>

<template>
  <div class="flex flex-col gap-0">
    <div
      v-for="(item, i) in items"
      :key="i"
      class="flex gap-4 relative"
    >
      <div class="flex flex-col items-center">
        <div
          class="w-3 h-3 rounded-full shrink-0 mt-1.5"
          style="background: #0070f3"
        />
        <div
          v-if="i < items.length - 1"
          class="w-px flex-1"
          style="background: rgba(255,255,255,0.1)"
        />
      </div>
      <div class="pb-6">
        <h4 class="text-sm font-semibold m-0 mb-1">{{ item.title }}</h4>
        <p class="text-xs opacity-50 m-0 leading-relaxed">{{ item.description }}</p>
      </div>
    </div>
  </div>
</template>
