<template>
  <!-- Footer -->
  <div
    v-if="$nav.currentPage !== 1 && $nav.currentPage !== $nav.total"
    class="absolute bottom-0 left-0 right-0 flex items-center justify-between px-10 py-4"
    style="font-size: 0.62rem; color: rgba(10, 10, 10, 0.42); z-index: 2; letter-spacing: 0.08em"
  >
    <span style="text-transform: uppercase">権代颯士 — 2026/06/26 LT</span>
    <span style="font-variant-numeric: tabular-nums">{{ String($nav.currentPage).padStart(2, '0') }} / {{ String($nav.total).padStart(2, '0') }}</span>
  </div>
</template>
