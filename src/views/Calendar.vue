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
    <!-- Сообщение о приветствии для неаутентифицированных пользователей -->
    <div class="welcome-message" v-if="!isAuthenticated">
      <p>Добро пожаловать! Зарегистрируйтесь или войдите, чтобы начать использовать наше приложение.</p>
    </div>

    <!-- Контейнер для досок и задач -->
    <div v-if="isAuthenticated" class="desks-container">
      <!-- Если нет досок и задач -->
      <div v-if="desks.length === 0">
        <p class="poprob">Пока что у вас нет досок и задач, попробуйте добавить их!</p>
      </div>
      <!-- Вывод досок и задач -->
      <div v-else class="desks-wrapper">
        <div v-for="desk in desks" :key="desk.id" class="desk-column">
          <!-- Заголовок доски -->
          <div class="desk-header">
            <h3>{{ desk.name }}</h3>
            <button v-if="!desk.completed" @click="openEditModal(desk)" class="edit-button">Изменить</button>
          </div>
          <!-- Список задач -->
          <ul class="task-list">
            <li v-if="desk.tasks.length === 0">Задач нет</li>
            <li v-for="task in sortedTasks(desk.tasks)" :key="task.id" :class="{ completed: task.completed, highPriority: task.highPriority }" @click="!task.completed && openTaskDetailModal(task)">
              <span>{{ task.name }}</span>
              <button v-if="!task.completed" @click.stop="markAsCompleted(task)" class="complete-button">Выполнено</button>
              <button v-if="task.completed" @click.stop="markAsNotCompleted(task)" class="revert-button">Вернуть</button>
              <button @click.stop="togglePriority(task)" :class="{'priority-button': task.highPriority, 'normal-button': !task.highPriority}">✓</button>
            </li>
          </ul>
          <!-- Кнопка добавления задачи -->
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
    // Выход из приложения
    logout() {
      localStorage.removeItem('userToken');
      localStorage.removeItem('userEmail');
      this.$router.push('/');
      location.reload();
    },
    // Запрос на получение списка досок пользователя
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
          this.fetchTasks(); // Запрос задач после получения досок
        } else {
          console.error('Error fetching desks:', response.statusText);
        }
      } catch (error) {
        console.error('Error fetching desks:', error);
      }
    },
    // Запрос на получение списка задач пользователя
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
          // Преобразуем highPriority в числовое значение (2 для высокого приоритета, 0 для обычного)
          const tasks = data.data.tasks.map(task => ({
            ...task,
            highPriority: task.highPriority ? 2 : 0
          }));
          this.assignTasksToDesks(tasks); // Распределение задач по доскам
        } else {
          console.error('Error fetching tasks:', response.statusText);
        }
      } catch (error) {
        console.error('Error fetching tasks:', error);
      }
    },
    // Распределение задач по соответствующим доскам
    assignTasksToDesks(tasks) {
      tasks.forEach(task => {
        const desk = this.desks.find(d => d.id === task.desk_id);
        if (desk) {
          // Проверяем, есть ли уже сохраненное значение приоритета в localStorage
          const storedPriority = localStorage.getItem(`taskPriority_${task.id}`);
          if (storedPriority !== null) {
            // Если есть, устанавливаем его в task.highPriority
            task.highPriority = parseInt(storedPriority, 10);
          } else {
            // Иначе сохраняем текущее значение приоритета в localStorage
            localStorage.setItem(`taskPriority_${task.id}`, task.highPriority.toString());
          }
          desk.tasks.push(task);
        }
      });
      this.sortTasks(); // Сортировка задач после распределения
    },
    // Сортировка задач на досках
    sortTasks() {
      this.desks.forEach(desk => {
        desk.tasks.sort((a, b) => b.highPriority - a.highPriority || new Date(a.created_at) - new Date(b.created_at));
      });
    },
    // Пометить задачу как выполненную
    async markAsCompleted(task) {
      try {
        const url = `${thisUrl()}/tasks/edit/${task.id}`;
        const userToken = localStorage.getItem('userToken');
        const response = await fetch(url, {
          method: 'PATCH',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${userToken}`
          },
          body: JSON.stringify({completed: true})
        });
        if (response.ok) {
          task.completed = true;
          console.log('Task marked as completed:', task);
          this.sortTasks();
        } else {
          const errorData = await response.json();
          console.error('Error marking task as completed:', errorData);
        }
      } catch (error) {
        console.error('Error marking task as completed:', error);
      }
    },
// Пометить задачу как не выполненную
    async markAsNotCompleted(task) {
      try {
        const url = `${thisUrl()}/tasks/edit/${task.id}`;
        const userToken = localStorage.getItem('userToken');
        const response = await fetch(url, {
          method: 'PATCH',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${userToken}`
          },
          body: JSON.stringify({ completed: false })
        });
        if (response.ok) {
          task.completed = false;
          console.log('Task marked as not completed:', task);
          this.sortTasks(); // После пометки задачи как не выполненной, производим сортировку
        } else {
          const errorData = await response.json();
          console.error('Error marking task as not completed:', errorData);
        }
      } catch (error) {
        console.error('Error marking task as not completed:', error);
      }
    },

// Открыть модальное окно для редактирования доски
    openEditModal(desk) {
      this.selectedDesk = desk;
      this.showEditModal = true;
    },

// Закрыть модальное окно для редактирования доски
    closeEditModal() {
      this.selectedDesk = null;
      this.showEditModal = false;
    },

// Открыть модальное окно для добавления задачи
    openTaskModal(deskId) {
      this.selectedDeskId = deskId;
      this.showTaskModal = true;
    },

// Открыть модальное окно для детализации задачи
    openTaskDetailModal(task) {
      this.selectedTask = task;
      this.showTaskDetailModal = true;
    },

// Закрыть модальное окно для детализации задачи
    closeTaskDetailModal() {
      this.selectedTask = null;
      this.showTaskDetailModal = false;
    },

// Обработка обновления задачи
    handleTaskUpdated(updatedTask) {
      const desk = this.desks.find(d => d.id === updatedTask.desk_id);
      if (desk) {
        const taskIndex = desk.tasks.findIndex(task => task.id === updatedTask.id);
        if (taskIndex !== -1) {
          desk.tasks.splice(taskIndex, 1, updatedTask);
        }
      }
      this.sortTasks(); // После обновления задачи, производим сортировку
    },

// Обработка удаления задачи
    handleTaskDeleted(deletedTaskId) {
      this.desks.forEach(desk => {
        const taskIndex = desk.tasks.findIndex(task => task.id === deletedTaskId);
        if (taskIndex !== -1) {
          desk.tasks.splice(taskIndex, 1);
        }
      });
      this.sortTasks(); // После удаления задачи, производим сортировку
    },

// Переключение приоритета задачи
    async togglePriority(task) {
      try {
        const url = `${thisUrl()}/tasks/edit/${task.id}`;
        const userToken = localStorage.getItem('userToken');
        const response = await fetch(url, {
          method: 'PATCH',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${userToken}`
          },
          body: JSON.stringify({ highPriority: task.highPriority === 2 ? 0 : 2 }) // 2 для высокого приоритета, 0 для обычного
        });
        if (response.ok) {
          task.highPriority = task.highPriority === 2 ? 0 : 2; // Обновление локального состояния задачи
          console.log('Task priority toggled:', task);
          this.sortTasks(); // После переключения приоритета, производим сортировку
        } else {
          const errorData = await response.json();
          console.error('Error toggling task priority:', errorData);
        }
      } catch (error) {
        console.error('Error toggling task priority:', error);
      }
    },

// Сортировка задач
    sortedTasks(tasks) {
      return tasks.slice().sort((a, b) => {
        if (a.highPriority > b.highPriority) return -1;
        if (a.highPriority < b.highPriority) return 1;
        // Если важность одинакова, сортировать по времени создания
        return new Date(a.created_at) - new Date(b.created_at);
      });
    },

// Открыть модальное окно для добавления новой доски
    openModal() {
      this.showModal = true;
    }
  },
// Жизненный цикл created для инициализации данных
  created() {
    if (this.isAuthenticated) {
      this.fetchDesks(); // При загрузке компонента, запрашиваем доски пользователя
    }
  }
};
</script>


<style scoped>
/* Оставляем CSS без изменений */
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

.task-list li.highPriority {
  font-weight: bold;
  color: red;
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

.priority-button {
  background-color: red;
  border: none;
  color: white;
  padding: 5px 10px;
  border-radius: 4px;
  cursor: pointer;
  margin-left: 10px;
  transition: background-color 0.2s;
}

.priority-button:hover {
  background-color: darkred;
}

.normal-button {
  background-color: gray;
  border: none;
  color: white;
  padding: 5px 10px;
  border-radius: 4px;
  cursor: pointer;
  margin-left: 10px;
  transition: background-color 0.2s;
}

.normal-button:hover {
  background-color: darkgray;
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
</style>