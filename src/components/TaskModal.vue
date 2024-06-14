<template>
  <div class="modal-overlay" @click.self="close">
    <div class="modal-content">
      <h2>Добавить задачу</h2>
      <form @submit.prevent="submit">
        <div class="form-group">
          <label for="task-title">Название задачи</label>
          <input type="text" id="task-title" v-model="name" required class="input-field" />
        </div>
        <div class="form-group">
          <label for="task-date">Дата примерного конца срока задачи</label>
          <input type="date" id="task-date" v-model="date" required class="input-field" />
        </div>
        <div class="form-group">
          <label for="task-img">Изображение (необязательно)</label>
          <input type="file" id="task-img" @change="onImageChange" class="input-file" />
        </div>
        <div class="button-group">
          <button type="submit" class="btn btn-save" :disabled="loading">
            {{ loading ? 'Сохранение...' : 'Сохранить' }}
          </button>
          <button type="button" class="btn btn-cancel" @click="close">Отмена</button>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
import { thisUrl } from "@/utils/api";

export default {
  name: 'TaskModal',
  props: {
    deskId: Number
  },
  data() {
    return {
      name: '',
      date: '',
      img: null,
      loading: false
    };
  },
  methods: {
    close() {
      this.$emit('close');
    },
    onImageChange(event) {
      this.img = event.target.files[0];
    },
    async submit() {
      try {
        this.loading = true;

        const url = `${thisUrl()}/tasks/create`;
        const userToken = localStorage.getItem('userToken');
        if (!userToken) {
          throw new Error('User token not found');
        }

        const formData = new FormData();
        formData.append('name', this.name);
        formData.append('date', this.date);
        formData.append('desk_id', this.deskId);
        if (this.img) {
          formData.append('img', this.img);
        }

        const response = await fetch(url, {
          method: 'POST',
          headers: {
            'Authorization': `Bearer ${userToken}`
          },
          body: formData
        });

        if (response.ok) {
          const responseData = await response.json();
          console.log('Task created:', responseData);
          this.$emit('task-created');
          this.close();
        } else {
          const errorData = await response.json();
          console.error('Error creating task:', errorData);
          // Handle error
        }
      } catch (error) {
        console.error('Error creating task:', error);
      } finally {
        this.loading = false;
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
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background-color: #ffffff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  max-width: 400px;
  width: 100%;
}

.modal-content h2 {
  font-size: 1.5rem;
  margin-bottom: 20px;
  text-align: center;
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: bold;
}

.input-field {
  width: 95%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 1rem;
}

.input-file {
  width: 95%;
  padding: 8px;
  font-size: 1rem;
}

.button-group {
  display: flex;
  justify-content: flex-end;
}

.btn {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
}

.btn-save {
  background-color: #2980b9;
  color: white;
  margin-right: 10px;
}

.btn-cancel {
  background-color: #a52a2a;
  color: white;
}

.btn:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}
</style>
