<script setup lang="ts">
defineProps<{
  title: string
  subtitle?: string
}>()
</script>

<template>
  <div class="demo-container">
    <div class="demo-ring">
      <div class="demo-ring-inner">
        <div class="play-icon" />
      </div>
    </div>
    <div class="demo-text">
      <h2 class="demo-title">{{ title }}</h2>
      <p v-if="subtitle" class="demo-subtitle">{{ subtitle }}</p>
    </div>
  </div>
</template>

<style scoped>
.demo-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 2rem;
  min-height: 60vh;
}

.demo-ring {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  padding: 1px;
  background: rgba(255, 255, 255, 0.15);
}

.demo-ring-inner {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  background: #0a0a0f;
  display: flex;
  align-items: center;
  justify-content: center;
}

.play-icon {
  width: 0;
  height: 0;
  border-style: solid;
  border-width: 18px 0 18px 32px;
  border-color: transparent transparent transparent #fff;
  margin-left: 6px;
}

.demo-title {
  font-size: 2rem;
  font-weight: 700;
  margin: 0;
  color: #fff;
  letter-spacing: -0.02em;
}

.demo-subtitle {
  margin: 0.5rem 0 0;
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.4);
  text-align: center;
}

.demo-text {
  text-align: center;
}
</style>
