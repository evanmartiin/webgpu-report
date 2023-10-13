<script setup>
defineProps({
  name: String,
  description: String,
})
</script>

<template>
  <div>
    <h4>{{ name }}</h4>
    <p>{{ description }}</p>
  </div>
</template>

<style scoped>
div {
  border-radius: 10px;
  position: fixed;
  left: 10px;
  bottom: 10px;
  width: 350px;
  border-radius: 10px;
  border: 2px solid #04AA6D;
  background-color: white;
  overflow: hidden;
  overflow-wrap: anywhere;
}
h4 {
  color: white;
  background-color: #04AA6D;
  padding: 10px 15px;
  text-align: center;
}
p {
  padding: 10px 15px;
}
</style>
