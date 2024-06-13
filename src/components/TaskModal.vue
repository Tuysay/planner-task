<template>
  <div class="modal-overlay" @click.self="close">
    <div class="modal-content">
      <h2>Добавить задачу</h2>
      <form @submit.prevent="submit">
        <div class="form-group">
          <label for="task-title">Название задачи</label>
          <input type="text" id="task-title" v-model="name" required />
        </div>
        <div class="form-group">
          <label for="task-date">Дата начала</label>
          <input type="date" id="task-date" v-model="date" required />
        </div>
<!--        <div class="form-group">-->
<!--          <label for="task-completed-date">Дата завершения</label>-->
<!--          <input type="number" id="task-completed-date" v-model="expired" />-->
<!--        </div>-->
<!--        <div class="form-group">-->
<!--          <label for="task-img">Изображение (необязательно)</label>-->
<!--          <input type="file" id="task-img" @change="onImageChange" />-->
<!--        </div>-->
        <button type="submit" class="btn-save">Сохранить</button>
        <button type="button" class="btn-cancel" @click="close">Отмена</button>
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
      date: '',  // Дата начала
      // expired: '',  // Дата завершения
      img: null  // Изображение
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
        if (!this.deskId) {
          console.error('deskId is not provided');
          return;
        }

        const url = `${thisUrl()}/tasks/create`;
        const userToken = localStorage.getItem('userToken');
        if (!userToken) {
          console.error('User token not found');
          throw new Error('User token not found');
        }

        const formData = new FormData();
        formData.append('name', this.name);
        formData.append('date', this.date);
        // formData.append('completed_date', this.expired);
        formData.append('desk_id', this.deskId);
        // if (this.img) {
        //   formData.append('img', this.img);
        // }

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
          this.$emit('task-created'); // Отправка события для обновления списка задач
          this.close();
        } else {
          const errorData = await response.json();
          console.error('Error creating task:', errorData);
        }
      } catch (error) {
        console.error('Error creating task:', error);
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
  background-color: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
}

.form-group input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.btn-save {
  background-color: #2980B9;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.btn-cancel {
  background-color: #A52A2A;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-left: 10px;
}
</style>
