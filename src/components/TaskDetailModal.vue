<template>
  <div class="modal-overlay" @click.self="close">
    <div class="modal-content">
      <h3>Детали задачи</h3>
      <div v-if="task">
        <div v-if="isEditing">
          <p>
            <strong>Название:</strong>
            <input v-model="editableTask.name" />
          </p>
          <p>
            <strong>Дата начала:</strong>
            <input type="date" v-model="editableTask.date" />
          </p>
          <p>
            <strong>Завершено:</strong>
            <input type="checkbox" v-model="editableTask.completed" @change="debouncedUpdateTaskCompletion" />
          </p>
          <button @click="saveChanges">Сохранить</button>
          <button @click="cancelEditing">Отмена</button>
        </div>
        <div v-else>
          <p><strong>Название:</strong> {{ task.name }}</p>
          <p><strong>Дата начала:</strong> {{ formatDate(task.date) }}</p>
          <p>
            <strong>Завершено:</strong>
            <input type="checkbox" :checked="task.completed" @change="debouncedUpdateTaskCompletion" />
          </p>
          <p v-if="task.img"><strong>Изображение:</strong> <img :src="task.img" alt="Изображение задачи" /></p>
          <button @click="enableEditing">Изменить</button>
          <button @click="deleteTask">Удалить</button>
        </div>
      </div>
      <button @click="close">Закрыть</button>
    </div>
  </div>
</template>

<script>
import { debounce } from 'lodash'; // добавляем lodash для дебаунса
import { thisUrl } from '@/utils/api';

export default {
  props: {
    task: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      isEditing: false,
      editableTask: { ...this.task }
    };
  },
  methods: {
    close() {
      this.$emit('close');
    },
    formatDate(dateString) {
      if (!dateString) return '';
      const options = { year: 'numeric', month: 'long', day: 'numeric' };
      return new Date(dateString).toLocaleDateString(undefined, options);
    },
    enableEditing() {
      this.isEditing = true;
      this.editableTask = { ...this.task };
    },
    cancelEditing() {
      this.isEditing = false;
      this.editableTask = { ...this.task };
    },
    async saveChanges() {
      try {
        const url = `${thisUrl()}/tasks/${this.task.id}`;
        const userToken = localStorage.getItem('userToken');
        if (!userToken) {
          console.error('User token not found');
          return;
        }

        const response = await fetch(url, {
          method: 'PATCH',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${userToken}`
          },
          body: JSON.stringify(this.editableTask)
        });

        if (response.ok) {
          const updatedTask = await response.json();
          this.task = { ...updatedTask.data };
          this.isEditing = false;
          console.log('Task updated:', updatedTask);
        } else {
          const errorData = await response.json();
          console.error('Error updating task:', errorData);
        }
      } catch (error) {
        console.error('Error updating task:', error);
      }
    },
    async updateTaskCompletion() {
      try {
        const url = `${thisUrl()}/tasks/edit/${this.task.id}`;
        const userToken = localStorage.getItem('userToken');
        if (!userToken) {
          console.error('User token not found');
          return;
        }

        const response = await fetch(url, {
          method: 'PATCH',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${userToken}`
          },
          body: JSON.stringify({ completed: !this.task.completed }) // переключаем состояние
        });

        if (response.ok) {
          this.task.completed = !this.task.completed; // обновляем локальное состояние
          const updatedTask = await response.json();
          console.log('Task updated:', updatedTask);
        } else {
          const errorData = await response.json();
          console.error('Error updating task:', errorData);
        }
      } catch (error) {
        console.error('Error updating task:', error);
      }
    },
    async deleteTask() {
      try {
        const url = `${thisUrl()}/tasks/delete/${this.task.id}`;
        const userToken = localStorage.getItem('userToken');
        if (!userToken) {
          console.error('User token not found');
          return;
        }

        const response = await fetch(url, {
          method: 'DELETE',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${userToken}`
          }
        });

        if (response.ok) {
          // Удаление задачи из родительского компонента или обновление списка задач
          this.$emit('task-deleted', this.task.id);
          console.log('Task deleted');
          this.close();
        } else {
          const errorData = await response.json();
          console.error('Error deleting task:', errorData);
        }
      } catch (error) {
        console.error('Error deleting task:', error);
      }
    }
  },
  created() {
    this.debouncedUpdateTaskCompletion = debounce(this.updateTaskCompletion, 300); // дебаунсинг
  }
};
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  width: 300px;
}

.modal-content h3 {
  margin-top: 0;
}

.modal-content img {
  max-width: 100%;
  height: auto;
  display: block;
}

.modal-content input[type="text"], .modal-content input[type="date"] {
  width: 100%;
  padding: 8px;
  margin: 5px 0;
  box-sizing: border-box;
}

button {
  margin-top: 10px;
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  background-color: #2980B9;
  color: white;
  cursor: pointer;
}

button:hover {
  background-color: #1F618D;
}
</style>
