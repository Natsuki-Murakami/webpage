<template>
  <footer class="slide-page-number">
    {{ $nav.currentPage }} / {{ $nav.total }}
  </footer>
</template>

<style scoped>
.slide-page-number {
  position: fixed;
  right: 1.5rem;
  bottom: 1rem;
  z-index: 50;
  padding: 0.2rem 0.7rem;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.9);
  color: #003366;
  font-size: 0.78rem;
  font-weight: 700;
  line-height: 1.4;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.12);
  pointer-events: none;
}
</style>
