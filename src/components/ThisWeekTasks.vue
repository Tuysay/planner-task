// ThisWeekTasks.vue
<template>
  <div class="task-list">
    <div class="task-item" v-for="(task, index) in tasks" :key="index" :style="{ backgroundColor: task.color }">
      <span>{{ task.title }}</span>
      <span>{{ task.count }}</span>
    </div>

  </div>
</template>

<script>
export default {
  props: ['tasks'],
  data() {
    return {

      thisWeekTasks: [
        { title: 'НА НЕДЕЛЕ', count: '0', color: '#1E90FF' },
        // Добавьте другие задачи на неделе
      ]

    };
  }
};
</script>

<style scoped>
.task-list {
  padding: 20px;
  //background-color: #34495E;
  display: flex;
  flex-direction: column;
  gap: 5px; /* Расстояние между задачами */
}
.task-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: white;
  padding: 15px 20px;
  margin-bottom: -25px;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s, box-shadow 0.3s;
}
.task-item:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 12px rgba(0, 0, 0, 0.2);
}

</style>
