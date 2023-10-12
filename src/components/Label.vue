<script setup>
defineProps({
  color: String,
})
</script>

<template>
  <div :style="{'background-color': color + '33', 'border': `2px solid ${color}`}">
    <slot />
  </div>
</template>

<style scoped>
div {
  padding: 10px 15px;
  border-radius: 10px;
  width: fit-content;
  display: flex;
  align-items: center;
  gap: 10px;
}
</style>
