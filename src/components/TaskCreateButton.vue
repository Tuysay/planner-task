<template>
  <button class="task-create-button" @click="$emit('click')">Добавить задачу</button>
</template>

<script>
export default {
  name: 'TaskCreateButton',
};
</script>

<style scoped>
.task-create-button {
  background-color: #3498db;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 10px 15px;
  cursor: pointer;
  transition: background-color 0.2s;
  margin-top: 10px;
}

.task-create-button:hover {
  background-color: #2980b9;
}
</style>
