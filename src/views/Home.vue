<!-- Home.vue -->
<template>
  <div>
    <nav class="navbar">
      <span v-if="!isAuthenticated">
        <router-link to="/login" class="navbar-item">Вход</router-link>
        <router-link to="/register" class="navbar-item">Регистрация</router-link>
      </span>
      <span v-else>
        <router-link to="/profile" class="navbar-item">Профиль</router-link>
        <router-link to="/calendar" class="navbar-item">Календарь</router-link>
        <router-link to="/" @click="logout" class="navbar-item">Выход</router-link>
        <span class="navbar-item">{{ userEmail }}</span> <!-- Отображение email пользователя -->
      </span>
    </nav>
    <div>
      <span v-if="!isAuthenticated">
        <TodayTasks :tasks="todayTasks" />
        <TomorrowTasks :tasks="tomorrowTasks" />
        <ThisWeekTasks :tasks="thisWeekTasks" />
        <ThisMonthTasks :tasks="thisMonthTasks" />
        <LaterTasks :tasks="laterTasks" />
      </span>
    </div>
    <div>
      <span v-if="isAuthenticated">
        <p class="poprob">
          Пока что у вас нет доск и задач, попробуйте добавить их!
        </p>
      </span>
    </div>
    <ButtonAdd @click="showModal = true" v-if="isAuthenticated" />
    <Modal v-if="showModal" @close="showModal = false" />
  </div>
</template>

<script>
import TodayTasks from '@/components/TodayTasks.vue';
import TomorrowTasks from '@/components/TomorrowTasks.vue';
import ThisWeekTasks from '@/components/ThisWeekTasks.vue';
import ThisMonthTasks from '@/components/ThisMonthTasks.vue';
import LaterTasks from '@/components/LaterTasks.vue';
import ButtonAdd from '@/components/ButtonAdd.vue';
import Modal from '@/components/Modal.vue';

export default {
  components: {
    TodayTasks,
    TomorrowTasks,
    ThisWeekTasks,
    ThisMonthTasks,
    LaterTasks,
    ButtonAdd,
    Modal
  },
  data() {
    return {
      showModal: false,
      todayTasks: [
        { title: 'СЕГОДНЯ', count: '0', color: '#2ECC71' }
        // Добавьте другие задачи для сегодня
      ],
      tomorrowTasks: [
        { title: 'ЗАВТРА', count: '0', color: '#3498DB' }
        // Добавьте другие задачи для завтра
      ],
      thisWeekTasks: [
        { title: 'НА НЕДЕЛЕ', count: '0', color: '#6495ED' }
        // Добавьте другие задачи на неделе
      ],
      thisMonthTasks: [
        { title: 'В ЭТОМ МЕСЯЦЕ', count: '0', color: '#7B68EE' }
        // Добавьте другие задачи в этом месяце
      ],
      laterTasks: [
        { title: 'ПОТОМ', count: '0', color: '#9B59B6' }
        // Добавьте другие задачи на будущее
      ]
    };
  },
  computed: {
    isAuthenticated() {
      return !!localStorage.getItem('userToken');
    },
    userEmail() {
      return localStorage.getItem('userEmail');
    }
  },
  methods: {
    logout() {
      localStorage.removeItem('userToken');
      localStorage.removeItem('userEmail'); // Удаление email из localStorage при выходе
      this.$router.push('/');
      location.reload();
    }
  }
};
</script>

<style scoped>
.navbar {
  display: flex;
  justify-content: center;
  background-color: #2C3E50;
  padding: 10px;
}
.navbar-item {
  color: orange;
  margin: 0 10px;
  text-decoration: none;
}
.navbar-item:hover {
  text-decoration: underline;
}
.poprob {
  color: orange;
  text-decoration: none;
  font-size: 36px;
  margin-top: 5%;
  margin-left: 5%;
}
</style>
