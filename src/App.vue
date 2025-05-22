<script setup lang="ts">
import taskItem from './components/taskitem.vue';

</script>

<template>
  <div>
    <div class="">
      <h1 class="text-3xl font-bold">Vue To-Do</h1>
    </div>
    <div class="todo-container p-8">
      <taskItem />
    </div>
  </div>

</template>

<style scoped></style>
