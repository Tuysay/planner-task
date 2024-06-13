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
        <span class="navbar-item">{{ userEmail }}</span>
      </span>
    </nav>
    <!-- Приветственное сообщение -->
    <div class="welcome-message" v-if="!isAuthenticated">
      <p>Добро пожаловать! Зарегистрируйтесь или войдите, чтобы начать использовать наше приложение.</p>
    </div>
    <div v-if="isAuthenticated" class="desks-container">
      <div v-if="desks.length === 0">
        <p class="poprob">Пока что у вас нет досок и задач, попробуйте добавить их!</p>
      </div>
      <div v-else class="desks-wrapper">
        <div v-for="desk in desks" :key="desk.id" class="desk-column">
          <div class="desk-header">
            <h3>{{ desk.name }}</h3>
            <button @click="openEditModal(desk)" class="edit-button">Изменить</button>
          </div>
          <ul class="task-list">
            <li v-if="desk.tasks.length === 0">Задач нет</li>
            <li v-for="task in desk.tasks" :key="task.id" @click="openTaskDetailModal(task)">{{ task.name }}</li>
          </ul>
          <TaskCreateButton @click="openTaskModal(desk.id)" />
        </div>
      </div>
    </div>
    <ButtonAdd @click="openModal" v-if="isAuthenticated" class="add-button" />
    <Modal v-if="showModal" :deskId="selectedDeskId" @close="showModal = false" @task-created="fetchDesks" />
    <EditModal v-if="showEditModal" :desk="selectedDesk" @close="closeEditModal" @desk-updated="fetchDesks" />
    <TaskModal v-if="showTaskModal" :deskId="selectedDeskId" @close="showTaskModal = false" @task-created="fetchDesks" />
    <TaskEditModal v-if="showTaskEditModal" :task="selectedTask" @close="closeTaskEditModal" @task-updated="fetchDesks" />
    <TaskDetailModal v-if="showTaskDetailModal" :task="selectedTask" @close="closeTaskDetailModal" />
  </div>
</template>

<script>
import ButtonAdd from '@/components/ButtonAdd.vue';
import Modal from '@/components/Modal.vue';
import EditModal from '@/components/EditModal.vue';
import TaskModal from '@/components/TaskModal.vue';
import TaskEditModal from '@/components/TaskEditModal.vue';
import TaskDetailModal from '@/components/TaskDetailModal.vue';
import TaskCreateButton from '@/components/TaskCreateButton.vue';
import { thisUrl } from '@/utils/api';

export default {
  components: {
    ButtonAdd,
    Modal,
    EditModal,
    TaskModal,
    TaskEditModal,
    TaskDetailModal,
    TaskCreateButton,
  },
  data() {
    return {
      showModal: false,
      showEditModal: false,
      desks: [],
      selectedDesk: null,
      selectedDeskId: null,
      showTaskModal: false,
      showTaskEditModal: false,
      showTaskDetailModal: false,
      selectedTask: null,
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
      localStorage.removeItem('userEmail');
      this.$router.push('/');
      location.reload();
    },
    async fetchDesks() {
      try {
        const url = thisUrl() + "/desks";
        const userToken = localStorage.getItem('userToken');
        const response = await fetch(url, {
          method: 'GET',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${userToken}`
          }
        });
        if (response.ok) {
          const data = await response.json();
          console.log('Fetched desks:', data);
          this.desks = data.data.desks.map(desk => ({
            ...desk,
            tasks: []
          }));
          this.fetchTasks();
        } else {
          console.error('Error fetching desks:', response.statusText);
        }
      } catch (error) {
        console.error('Error fetching desks:', error);
      }
    },
    async fetchTasks() {
      try {
        const url = thisUrl() + "/tasks";
        const userToken = localStorage.getItem('userToken');
        const response = await fetch(url, {
          method: 'GET',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${userToken}`
          }
        });
        if (response.ok) {
          const data = await response.json();
          console.log('Fetched tasks:', data);
          this.assignTasksToDesks(data.data.tasks);
        } else {
          console.error('Error fetching tasks:', response.statusText);
        }
      } catch (error) {
        console.error('Error fetching tasks:', error);
      }
    },
    assignTasksToDesks(tasks) {
      tasks.forEach(task => {
        const desk = this.desks.find(d => d.id === task.desk_id);
        if (desk) {
          desk.tasks.push(task);
        }
      });
    },
    openEditModal(desk) {
      this.selectedDesk = desk;
      this.showEditModal = true;
    },
    closeEditModal() {
      this.selectedDesk = null;
      this.showEditModal = false;
    },
    openTaskModal(deskId) {
      this.selectedDeskId = deskId;
      this.showTaskModal = true;
    },
    openTaskDetailModal(task) {
      this.selectedTask = task;
      this.showTaskDetailModal = true;
    },
    closeTaskDetailModal() {
      this.selectedTask = null;
      this.showTaskDetailModal = false;
    },
    openTaskEditModal(task) {
      this.selectedTask = task;
      this.showTaskEditModal = true;
    },
    closeTaskEditModal() {
      this.selectedTask = null;
      this.showTaskEditModal = false;
    },
    openModal() {
      this.showModal = true;
    }
  },
  created() {
    if (this.isAuthenticated) {
      this.fetchDesks();
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

.desks-container {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  padding: 20px;
}

.desks-wrapper {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  justify-content: center;
  width: 100%;
  max-width: 1200px;
}

.desk-column {
  background-color: #fff;
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  width: 300px;
  color: #333;
  text-align: left;
  transition: transform 0.2s, box-shadow 0.2s;
  position: relative;
}

.desk-column:hover {
  transform: translateY(-5px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.desk-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.desk-column h3 {
  margin: 0;
  font-size: 20px;
  color: #333;
}

.edit-button {
  background-color: transparent;
  border: 1px solid #2980B9;
  color: #2980B9;
  padding: 5px 10px;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s, color 0.2s;
}

.edit-button:hover {
  background-color: #2980B9;
  color: white;
}

.task-list {
  list-style: none;
  padding: 0;
  margin-top: 10px;
}

.task-list li {
  padding: 5px 0;
  border-bottom: 1px solid #ddd;
}

.task-list li:last-child {
  border-bottom: none;
}

.welcome-message {
  color: orange;
  text-decoration: none;
  font-size: 36px;
  margin-top: 5%;
  margin-left: 5%;
}

.add-button {
  position: fixed;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  background-color: #e74c3c;
  color: white;
  border: none;
  border-radius: 50%;
  width: 60px;
  height: 60px;
  font-size: 36px;
  line-height: 60px;
  text-align: center;
  cursor: pointer;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  transition: background-color 0.3s, box-shadow 0.3s;
}

.add-button:hover {
  background-color: #800000;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
}
</style>
