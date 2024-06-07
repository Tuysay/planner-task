<template>
  <div class="modal-overlay" @click.self="close">
    <div class="modal-content">
      <h2>Добавить задачу</h2>
      <form @submit.prevent="submit">
        <div class="form-group">
          <label for="task-title">Название задачи</label>
          <input type="text" id="task-title" v-model="desks.name" required />
        </div>
        <button type="submit" class="btn-save">Сохранить</button>
        <button type="button" class="btn-cancel" @click="close">Отмена</button>
      </form>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Modal',
  props: {
    show: Boolean
  },
  data() {
    return {
      desks: {
        name: ''
      }
    };
  },
  methods: {
    close() {
      this.$emit('close');
    },
    submit() {
      const taskData = {
        name: this.desks.name
      };

      fetch('/desks/create', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          Authorization: `Bearer ${localStorage.getItem('userToken')}`
        },
        body: JSON.stringify(taskData)
      })
          .then(response => {
            if (!response.ok) {
              throw new Error('Network response was not ok');
            }
            return response.json();
          })
          .then(data => {
            console.log('Task saved:', data);
            this.close();
          })
          .catch(error => {
            console.error('Error saving task:', error);
          });
    }
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
  background: #696969; /* Изменено на черный */
  padding: 20px;
  border-radius: 8px;
  width: 400px;
  max-width: 90%;
}
.form-group {
  margin-bottom: 20px; /* Увеличили отступ для улучшения визуального восприятия */
}
.form-group label {
  display: block;
  margin-bottom: 8px; /* Уменьшили отступ между меткой и полем ввода */
  color: #FFFAFA; /* Изменили цвет текста меток на белый */
}
.form-group input {
  width: 100%;
  padding: 10px; /* Увеличили отступы вокруг текста в полях ввода */
  box-sizing: border-box;
  border: 1px solid #ccc; /* Добавили рамку для поля ввода */
  border-radius: 10px; /* Закруглили углы поля ввода */
  background-color: #808080; /* Изменили цвет фона поля ввода */
  color: white; /* Изменили цвет текста в полях ввода на белый */
}
.btn-save {
  background-color: #2F4F4F;
  color: white;
  padding: 12px 20px; /* Увеличили отступы кнопок для лучшей акцентации */
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
.btn-cancel {
  background-color: #A52A2A;
  color: white;
  padding: 12px 20px; /* Увеличили отступы кнопок для лучшей акцентации */
  border: none;
  border-radius: 5px;
  cursor: pointer;
  margin-left: 10px;
}
</style>
