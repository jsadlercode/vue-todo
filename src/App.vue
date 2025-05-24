<script setup lang="ts">

import TaskList from './components/TaskList.vue'



</script>

<template>
  <div class="flex flex-col">
    <div class="">
      <h1 class="text-3xl font-bold">Vue To-Do</h1>
    </div>
    <div class="todo-container p-8 mx-auto">
      <TaskList class=""/>
    </div>
  </div>

</template>

<style scoped></style>
