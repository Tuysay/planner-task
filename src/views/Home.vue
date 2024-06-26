<template>
  <div>
    <nav class="navbar">
      <span v-if="!isAuthenticated">
        <router-link to="/login" class="navbar-item">Вход</router-link>
        <router-link to="/register" class="navbar-item">Регистрация</router-link>
      </span>
      <span v-else>
        <router-link to="/profile" class="navbar-item">Профиль</router-link>
        <router-link to="/" @click="logout" class="navbar-item">Выход</router-link>
        <span class="navbar-item">{{ userEmail }}</span>
      </span>
    </nav>
    <div class="welcome-message" v-if="!isAuthenticated">
      <p>Добро пожаловать! Зарегистрируйтесь или войдите, чтобы начать использовать наше приложение.</p>
    </div>
    <div v-if="isAuthenticated" class="desks-container">
      <div v-if="desks.length === 0">
        <p class="poprob">Пока что у вас нет досок и задач, попробуйте добавить их!</p>
      </div>
      <div v-else class="desks-wrapper">
        <div v-for="desk in sortedDesks" :key="desk.id" class="desk-column">
          <div class="desk-header">
            <h3>{{ desk.name }}</h3>
            <button v-if="!desk.completed" @click="openEditModal(desk)" class="edit-button">Изменить</button>
          </div>
          <ul class="task-list">
            <li v-if="desk.tasks.length === 0">Задач нет</li>
            <li v-for="task in desk.tasks" :key="task.id" :class="{ completed: task.completed }" @click="!task.completed && openTaskDetailModal(task)">
              <span>{{ task.name }}</span>
              <button v-if="!task.completed" @click.stop="markAsCompleted(task)" class="complete-button">Выполнено</button>
            </li>
          </ul>
          <div class="max-tasks-reached-message" v-if="maxTasksReachedDeskId === desk.id">
            <p>Доска {{ desk.name }} уже содержит максимальное количество задач (5). Новая задача не может быть добавлена.</p>
            <button @click="confirmDeleteDesk(desk)" class="delete-desk-button">Удалить доску</button>
          </div>
          <TaskCreateButton @click="openTaskModal(desk.id)" />
        </div>
      </div>
    </div>
    <ButtonAdd @click="openModal" v-if="isAuthenticated" class="add-button" />
    <Modal v-if="showModal" :deskId="selectedDeskId" @close="showModal = false" @task-created="fetchDesks" />
    <EditModal v-if="showEditModal" :desk="selectedDesk" @close="closeEditModal" @desk-updated="fetchDesks" />
    <TaskModal v-if="showTaskModal" :deskId="selectedDeskId" @close="showTaskModal = false" @task-created="fetchDesks" />
    <TaskDetailModal v-if="showTaskDetailModal" :task="selectedTask" @close="closeTaskDetailModal" @task-updated="handleTaskUpdated" @task-deleted="handleTaskDeleted" />
  </div>
</template>

<script>
import ButtonAdd from '@/components/ButtonAdd.vue';
import Modal from '@/components/Modal.vue';
import EditModal from '@/components/EditModal.vue';
import TaskModal from '@/components/TaskModal.vue';
import TaskDetailModal from '@/components/TaskDetailModal.vue';
import TaskCreateButton from '@/components/TaskCreateButton.vue';
import { thisUrl } from '@/utils/api';

export default {
  components: {
    ButtonAdd,
    Modal,
    EditModal,
    TaskModal,
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
      showTaskDetailModal: false,
      selectedTask: null,
      maxTasksReachedDeskId: null,
    };
  },
  computed: {
    isAuthenticated() {
      return !!localStorage.getItem('userToken');
    },
    userEmail() {
      return localStorage.getItem('userEmail');
    },
    sortedDesks() {
      return this.desks.map(desk => {
        return {
          ...desk,
          tasks: desk.tasks.slice().sort((a, b) => a.completed - b.completed)
        };
      });
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

          if (new Date(task.created_at) >= new Date(task.date)) {
            task.completed = 1;
            this.markAsCompleted(task);
          }

          if (desk.tasks.length < 5) {
            desk.tasks.push(task);
          } else {
            this.maxTasksReachedDeskId = desk.id;
            setTimeout(() => {
              if (this.maxTasksReachedDeskId === desk.id) {
                this.maxTasksReachedDeskId = null;
              }
            }, 5000);
          }
        }
      });
    },
    async markAsCompleted(task) {
      try {
        const url = `${thisUrl()}/tasks/complete/${task.id}`;
        const userToken = localStorage.getItem('userToken');
        const response = await fetch(url, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${userToken}`
          },
          body: JSON.stringify({ completed: 1 })
        });
        if (response.ok) {
          task.completed = 1;
          console.log('Task marked as completed:', task);
        } else {
          const errorData = await response.json();
          console.error('Error marking task as completed:', errorData);
        }
      } catch (error) {
        console.error('Error marking task as completed:', error);
      }
    },
    confirmDeleteDesk(desk) {
      if (confirm(`Вы действительно хотите удалить доску "${desk.name}"?`)) {
        this.deleteDesk(desk);
      }
    },
    async deleteDesk(desk) {
      try {
        const url = `${thisUrl()}/desks/delete/${desk.id}`;

        const userToken = localStorage.getItem('userToken');
        const response = await fetch(url, {
          method: 'DELETE',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${userToken}`
          }
        });

        if (response.ok) {
          this.fetchDesks(); // Обновляем список досок после удаления
        } else {
          console.error('Error deleting desk:', response.statusText);
        }
      } catch (error) {
        console.error('Error deleting desk:', error);
      }
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
    handleTaskUpdated(updatedTask) {
      const desk = this.desks.find(d => d.id === updatedTask.desk_id);
      if (desk) {
        const taskIndex = desk.tasks.findIndex(task => task.id === updatedTask.id);
        if (taskIndex !== -1) {
          desk.tasks.splice(taskIndex, 1, updatedTask);
        }
      }
    },
    handleTaskDeleted(deletedTaskId) {
      this.desks.forEach(desk => {
        const taskIndex = desk.tasks.findIndex(task => task.id === deletedTaskId);
        if (taskIndex !== -1) {
          desk.tasks.splice(taskIndex, 1);
        }
      });
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
  display: flex;
  justify-content: space-between;
  align-items: center;
  transition: background-color 0.2s, color 0.2s;
}

.task-list li:last-child {
  border-bottom: none;
}

.task-list li.completed {
  text-decoration: line-through;
  color: #888;
}

.task-list li:hover {
  background-color: #f0f0f0;
  color: #000;
}

.complete-button {
  background-color: #27ae60;
  border: none;
  color: white;
  padding: 5px 10px;
  border-radius: 4px;
  cursor: pointer;
  margin-left: 10px;
  transition: background-color 0.2s;
}

.complete-button:hover {
  background-color: #219150;
}

.revert-button {
  background-color: #e67e22;
  border: none;
  color: white;
  padding: 5px 10px;
  border-radius: 4px;
  cursor: pointer;
  margin-left: 10px;
  transition: background-color 0.2s;
}

.revert-button:hover {
  background-color: #d35400;
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
  bottom: 10%;
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
.task-list li:hover {
  background-color: #f0f0f0;
  color: #000;
}
.add-button {
  position: fixed;
  bottom: 10%;
  left: 50%;
  transform: translateX(-50%);
}
.max-tasks-reached-message {
  background-color: #f2dede;
  color: #a94442;
  border: 1px solid #ebccd1;
  border-radius: 4px;
  padding: 10px;
  margin-top: 10px;
}

.delete-desk-button {
  background-color: #e74c3c;
  border: none;
  color: white;
  padding: 5px 10px;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 10px;
  transition: background-color 0.2s;
}

.delete-desk-button:hover {
  background-color: #800000;
}

.max-tasks-reached-message p {
  margin: 0;
  font-size: 14px;
}

</style>

