<template>
  <div class="modal-overlay" @click.self="close">
    <div class="modal-content">
      <h3>Детали задачи</h3>
      <div v-if="editableTask">
        <p>
          <strong>Название:</strong>
          <input v-model="editableTask.name" />
        </p>
        <p>
          <strong>Дата начала:</strong>
          <input type="date" v-model="editableTask.date" />
        </p>
        <p v-if="imageError">
          <strong>Ошибка загрузки изображения:</strong>
          <span>{{ imageError }}</span>
        </p>
        <p v-if="imageUrl && !imageError">
          <strong>Изображение:</strong><br>
          <img :src="imageUrl" alt="Task Image" class="task-image" />
        </p>
        <div class="button-group">
          <button class="btn-save" @click="saveChanges">Сохранить</button>
          <button class="btn-delete" @click="deleteTask">Удалить</button>
        </div>
      </div>
      <button class="btn-close" @click="close">Закрыть</button>
    </div>
  </div>
</template>

<script>
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
      editableTask: { ...this.task },
      imageUrl: null,
      imageError: null
    };
  },
  async created() {
    if (this.task.id) {
      this.imageUrl = await this.fetchTaskImage(this.task.id);
    }
  },
  methods: {
    close() {
      this.$emit('close');
    },
    async fetchTaskImage(taskId) {
      try {
        const url = `${thisUrl()}/tasks/image/${taskId}`;
        const response = await fetch(url);
        if (response.ok) {
          return response.url;
        } else if (response.status === 404) {
          this.imageError = 'Изображение не найдено';
          return null;
        } else {
          this.imageError = 'Ошибка загрузки изображения';
          return null;
        }
      } catch (error) {
        console.error('Error fetching task image:', error);
        this.imageError = 'Ошибка загрузки изображения';
        return null;
      }
    },
    async saveChanges() {
      try {
        const url = `${thisUrl()}/tasks/edit/${this.task.id}`;
        const userToken = localStorage.getItem('userToken');
        if (!userToken) {
          console.error('User token not found');
          return;
        }

        const updatedFields = {};
        if (this.editableTask.name !== this.task.name) updatedFields.name = this.editableTask.name;
        if (this.editableTask.date !== this.task.date) updatedFields.date = this.editableTask.date;

        const response = await fetch(url, {
          method: 'PATCH',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${userToken}`
          },
          body: JSON.stringify(updatedFields)
        });

        if (response.ok) {
          const updatedTask = await response.json();
          this.$emit('task-updated', updatedTask.data);
          console.log('Task updated:', updatedTask);
          this.close();
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
  animation: fadeIn 0.3s;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  width: 300px;
  animation: slideIn 0.3s;
}

@keyframes slideIn {
  from {
    transform: translateY(-10%);
  }
  to {
    transform: translateY(0);
  }
}

.modal-content h3 {
  margin-top: 0;
  font-size: 24px;
  color: #333;
  text-align: center;
}

.modal-content p {
  margin-bottom: 15px;
}

.modal-content input[type="text"],
.modal-content input[type="date"] {
  width: 100%;
  padding: 10px;
  margin: 5px 0;
  box-sizing: border-box;
  border: 1px solid #ccc;
  border-radius: 4px;
  transition: border-color 0.3s;
}

.modal-content input[type="text"]:focus,
.modal-content input[type="date"]:focus {
  border-color: #2980B9;
  outline: none;
}

.task-image {
  max-width: 100%;
  max-height: 150px;
  margin-top: 10px;
  border-radius: 4px;
  border: 1px solid #ccc;
}

.button-group {
  display: flex;
  justify-content: space-between;
}

button {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s;
}

.btn-save {
  background-color: #2980B9;
  color: white;
}

.btn-save:hover {
  background-color: #1F618D;
}

.btn-delete {
  background-color: #E74C3C;
  color: white;
}

.btn-delete:hover {
  background-color: #C0392B;
}

.btn-close {
  margin-top: 20px;
  background-color: #7F8C8D;
  color: white;
  display: block;
  width: 100%;
  text-align: center;
}

.btn-close:hover {
  background-color: #626E70;
}
</style>
