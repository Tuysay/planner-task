<!--&lt;!&ndash; TaskEditModal.vue &ndash;&gt;-->
<!--<template>-->
<!--  <div class="modal-overlay" @click.self="close">-->
<!--    <div class="modal-content">-->
<!--      <h2>Редактировать задачу</h2>-->
<!--      <form @submit.prevent="submit">-->
<!--        <div class="form-group">-->
<!--          <label for="task-title">Название задачи</label>-->
<!--          <input type="text" id="task-title" v-model="name" required />-->
<!--        </div>-->
<!--        <button type="submit" class="btn-save">Сохранить</button>-->
<!--        <button type="button" class="btn-cancel" @click="close">Отмена</button>-->
<!--      </form>-->
<!--    </div>-->
<!--  </div>-->
<!--</template>-->

<!--<script>-->
<!--import { thisUrl } from "@/utils/api";-->

<!--export default {-->
<!--  name: 'TaskEditModal',-->
<!--  props: {-->
<!--    task: Object-->
<!--  },-->
<!--  data() {-->
<!--    return {-->
<!--      name: this.task.name-->
<!--    };-->
<!--  },-->
<!--  methods: {-->
<!--    close() {-->
<!--      this.$emit('close');-->
<!--    },-->
<!--    async submit() {-->
<!--      try {-->
<!--        const url = thisUrl() + `/tasks/edit/${this.task.id}`;-->
<!--        const userToken = localStorage.getItem('userToken');-->
<!--        if (!userToken) {-->
<!--          console.error('User token not found');-->
<!--          throw new Error('User token not found');-->
<!--        }-->

<!--        const response = await fetch(url, {-->
<!--          method: 'PATCH',-->
<!--          headers: {-->
<!--            'Content-Type': 'application/json',-->
<!--            'Authorization': `Bearer ${userToken}`-->
<!--          },-->
<!--          body: JSON.stringify({ name: this.name }),-->
<!--        });-->

<!--        if (response.ok) {-->
<!--          await response.json();-->
<!--          this.$emit('task-updated'); // Отправка события для обновления списка задач-->
<!--          this.close();-->
<!--        } else {-->
<!--          console.error('Error updating task:', response.statusText);-->
<!--        }-->
<!--      } catch (error) {-->
<!--        console.error('Error updating task:', error);-->
<!--      }-->
<!--    }-->
<!--  }-->
<!--};-->
<!--</script>-->

<!--<style scoped>-->
<!--.modal-overlay {-->
<!--  position: fixed;-->
<!--  top: 0;-->
<!--  left: 0;-->
<!--  width: 100%;-->
<!--  height: 100%;-->
<!--  background-color: rgba(0, 0, 0, 0.5);-->
<!--  display: flex;-->
<!--  justify-content: center;-->
<!--  align-items: center;-->
<!--}-->
<!--.modal-content {-->
<!--  background-color: #fff;-->
<!--  padding: 20px;-->
<!--  border-radius: 8px;-->
<!--  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);-->
<!--  width: 100%;-->
<!--  max-width: 400px;-->
<!--}-->
<!--.form-group {-->
<!--  margin-bottom: 15px;-->
<!--}-->
<!--label {-->
<!--  display: block;-->
<!--  margin-bottom: 5px;-->
<!--}-->
<!--input[type="text"] {-->
<!--  width: 100%;-->
<!--  padding: 8px;-->
<!--  box-sizing: border-box;-->
<!--}-->
<!--.btn-save {-->
<!--  background-color: #2980B9;-->
<!--  color: #fff;-->
<!--  border: none;-->
<!--  padding: 10px 20px;-->
<!--  border-radius: 4px;-->
<!--  cursor: pointer;-->
<!--  transition: background-color 0.2s;-->
<!--}-->
<!--.btn-save:hover {-->
<!--  background-color: #1F618D;-->
<!--}-->
<!--.btn-cancel {-->
<!--  background-color: #e74c3c;-->
<!--  color: #fff;-->
<!--  border: none;-->
<!--  padding: 10px 20px;-->
<!--  border-radius: 4px;-->
<!--  cursor: pointer;-->
<!--  transition: background-color 0.2s;-->
<!--  margin-left: 10px;-->
<!--}-->
<!--.btn-cancel:hover {-->
<!--  background-color: #c0392b;-->
<!--}-->
<!--</style>-->
