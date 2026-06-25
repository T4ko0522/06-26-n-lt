<script setup lang="ts">
defineProps<{
  items: { label: string; value: string }[]
}>()
</script>

<template>
  <div class="grid gap-2">
    <div
      v-for="item in items"
      :key="item.label"
      class="tech-row"
    >
      <span class="tech-label">{{ item.label }}</span>
      <span class="tech-dot" />
      <span class="tech-value">{{ item.value }}</span>
    </div>
  </div>
</template>

<style scoped>
.tech-row {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.5rem 0;
  border-bottom: 1px solid rgba(255, 255, 255, 0.04);
}

.tech-row:last-child {
  border-bottom: none;
}

.tech-label {
  font-size: 0.7rem;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: rgba(255, 255, 255, 0.4);
  width: 7rem;
  flex-shrink: 0;
}

.tech-dot {
  flex: 1;
  height: 1px;
  background: repeating-linear-gradient(
    90deg,
    rgba(255, 255, 255, 0.08) 0px,
    rgba(255, 255, 255, 0.08) 2px,
    transparent 2px,
    transparent 6px
  );
}

.tech-value {
  font-size: 0.85rem;
  color: #fff;
}
</style>
