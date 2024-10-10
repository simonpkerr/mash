<script setup lang="ts">
defineProps<{
  gameIndex: number
  selectedIndex: number
  discarded?: boolean
}>()
</script>

<template>
  <span
    :class="{
      active: gameIndex === selectedIndex,
      discarded
    }"
    ><slot></slot
  ></span>
</template>

<style scoped>
span {
  font-size: 2rem;
  font-weight: 500;
  color: white;
  text-shadow: 0 0 0.5rem black;
  transition: all 0.3s ease;

  &.active {
    color: var(--secondary-color);
  }

  &.discarded {
    text-decoration: line-through;
    opacity: 0.5;
  }
}
</style>
