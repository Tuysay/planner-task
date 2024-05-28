<template>
  <div>
    <Navbar />
  </div>
    <div>
      <TodayTasks :tasks="todayTasks"></TodayTasks>
      <TomorrowTasks :tasks="tomorrowTasks"></TomorrowTasks>
      <ThisWeekTasks :tasks="thisWeekTasks"></ThisWeekTasks>
      <ThisMonthTasks :tasks="thisMonthTasks"></ThisMonthTasks>
      <LaterTasks :tasks="laterTasks"></LaterTasks>
    </div>
</template>

<script>
import TodayTasks from '@/components/TodayTasks.vue';
import TomorrowTasks from '@/components/TomorrowTasks.vue';
import ThisWeekTasks from '@/components/ThisWeekTasks.vue';
import ThisMonthTasks from '@/components/ThisMonthTasks.vue';
import LaterTasks from '@/components/LaterTasks.vue';

export default {
  components: {
    TodayTasks,
    TomorrowTasks,
    ThisWeekTasks,
    ThisMonthTasks,
    LaterTasks
  },
  data() {
    return {
      todayTasks: [
        { title: 'СЕГОДНЯ', count: '0/1', color: '#2ECC71' },
        // Добавьте другие задачи для сегодня
      ],
      tomorrowTasks: [
        { title: 'ЗАВТРА', count: '0', color: '#3498DB' },
        // Добавьте другие задачи для завтра
      ],
      thisWeekTasks: [
        { title: 'НА НЕДЕЛЕ', count: '0', color: '#9B59B6' },
        // Добавьте другие задачи на неделе
      ],
      thisMonthTasks: [
        { title: 'В ЭТОМ МЕСЯЦЕ', count: '0', color: '#8E44AD' },
        // Добавьте другие задачи в этом месяце
      ],
      laterTasks: [
        { title: 'ПОТОМ', count: '0', color: '#9B59B6' }
        // Добавьте другие задачи на будущее
      ]
    };
  }
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
