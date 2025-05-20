<script setup lang="ts">
import TodoList from './components/TodoList.vue';
</script>

<template>
  <div class="container">
    <!-- <p>Loading...</p>
    <Spinner :scale="3" /> -->
    <h1>Todo list</h1>
    <TodoList />
  </div>
</template>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
</style>
