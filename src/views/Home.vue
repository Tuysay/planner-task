<template>

    <div>
      <TodayTasks :tasks="todayTasks"></TodayTasks>
      <TomorrowTasks :tasks="tomorrowTasks"></TomorrowTasks>
      <ThisWeekTasks :tasks="thisWeekTasks"></ThisWeekTasks>
      <ThisMonthTasks :tasks="thisMonthTasks"></ThisMonthTasks>
      <LaterTasks :tasks="laterTasks"></LaterTasks>
    </div>
  <ButtonAdd  @click="showModal = true" />
  <Modal v-if="showModal" @close="showModal = false" />

</template>

<script>
import TodayTasks from '@/components/TodayTasks.vue';
import TomorrowTasks from '@/components/TomorrowTasks.vue';
import ThisWeekTasks from '@/components/ThisWeekTasks.vue';
import ThisMonthTasks from '@/components/ThisMonthTasks.vue';
import LaterTasks from '@/components/LaterTasks.vue';
import ButtonAdd from "@/components/ButtonAdd.vue";
import Modal from '../components/Modal.vue';

export default {
  components: {
    ButtonAdd,
    TodayTasks,
    TomorrowTasks,
    ThisWeekTasks,
    ThisMonthTasks,
    LaterTasks,
    Modal
  },
  data() {
    return {
      showModal: false,

      todayTasks: [
        { title: 'СЕГОДНЯ', count: '0', color: '#2ECC71' },
        // Добавьте другие задачи для сегодня
      ],
      tomorrowTasks: [
        { title: 'ЗАВТРА', count: '0', color: '#3498DB' },
        // Добавьте другие задачи для завтра
      ],
      thisWeekTasks: [
        { title: 'НА НЕДЕЛЕ', count: '0', color: '#6495ED' },
        // Добавьте другие задачи на неделе
      ],
      thisMonthTasks: [
        { title: 'В ЭТОМ МЕСЯЦЕ', count: '0', color: '#7B68EE' },
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

</style>
