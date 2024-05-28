// LaterTasks.vue
<template>
  <div>
    <div class="task-item" v-for="(task, index) in tasks" :key="index" :style="{ backgroundColor: task.color }">
      <span>{{ task.title }}</span>
      <span>{{ task.count }}</span>
    </div>
  </div>
</template>

<script>
export default {
  props: ['tasks']
};
</script>

<style scoped>
.task-list {
  padding: 10px;
  background-color: #34495E;
  height: calc(100vh - 50px);
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}
.task-item {
  display: flex;
  justify-content: space-between;
  color: white;
  padding: 15px;
  margin-bottom: 5px;
  border-radius: 5px;
}
.fab-container {
  display: flex;
  justify-content: center;
  gap: 15px;
  margin-top: auto;
}
.fab {
  background-color: #FF6B6B;
  border: none;
  border-radius: 50%;
  color: white;
  font-size: 24px;
  width: 50px;
  height: 50px;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0px 2px 5px rgba(0, 0, 0, 0.2);
}
</style>
