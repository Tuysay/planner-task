<template>
  <div class="modal-overlay" @click.self="close">
    <div class="modal-content">
      <h2>Добавить задачу</h2>
      <form @submit.prevent="submit">
        <div class="form-group">
          <label for="task-title">Название задачи</label>
          <input type="text" id="task-title" v-model="name" required />
        </div>
        <button type="submit" class="btn-save">Сохранить</button>
        <button type="button" class="btn-cancel" @click="close">Отмена</button>
      </form>
    </div>
  </div>
</template>

<script>
import { thisUrl } from "@/utils/api";

export default {
  name: 'Modal',
  props: {
    show: Boolean
  },
  data() {
    return {
      name: ''
    };
  },
  methods: {
    close() {
      this.$emit('close');
    },
    async submit() {
      try {
        const url = thisUrl() + "/desks/create";
        const userToken = localStorage.getItem('userToken');
        if (!userToken) {
          console.error('User token not found');
          throw new Error('User token not found');
        }

        const response = await fetch(url, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${userToken}`
          },
          body: JSON.stringify({ name: this.name }),
        });

        if (response.ok) {
          await response.json();
          this.$emit('task-created'); // Отправка события для обновления списка досок
          this.close();
        } else {
          console.error('Error creating task:', response.statusText);
        }
      } catch (error) {
        console.error('Error saving desks:', error);
      }
    },
  }
};
</script>

<style scoped>
.modal-content h2 {
  color: white;
}
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}
.modal-content {
  background: #696969;
  padding: 20px;
  border-radius: 8px;
  width: 400px;
  max-width: 90%;
}
.form-group {
  margin-bottom: 20px;
}
.form-group label {
  display: block;
  margin-bottom: 8px;
  color: #FFFAFA;
}
.form-group input {
  width: 100%;
  padding: 10px;
  box-sizing: border-box;
  border: 1px solid #ccc;
  border-radius: 10px;
  background-color: #808080;
  color: white;
}
.btn-save {
  background-color: #2F4F4F;
  color: white;
  padding: 12px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
.btn-cancel {
  background-color: #A52A2A;
  color: white;
  padding: 12px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  margin-left: 10px;
}
</style>
