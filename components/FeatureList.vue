<script setup lang="ts">
defineProps<{
  items: string[]
}>()
</script>

<template>
  <ul class="list-none p-0 m-0 grid gap-2.5">
    <li
      v-for="(item, i) in items"
      :key="i"
      class="feature-item"
    >
      <span class="feature-marker" />
      <span>{{ item }}</span>
    </li>
  </ul>
</template>

<style scoped>
.feature-item {
  display: flex;
  align-items: flex-start;
  gap: 0.75rem;
  font-size: 0.85rem;
  line-height: 1.6;
  color: rgba(255, 255, 255, 0.85);
}

.feature-marker {
  margin-top: 0.55rem;
  width: 5px;
  height: 5px;
  flex-shrink: 0;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.3);
}
</style>
