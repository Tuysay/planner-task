<!--изменение колонки-->
<template>
  <div class="modal-overlay" @click.self="close">
    <div class="modal-content">
      <h2>Изменить колонку</h2>
      <form @submit.prevent="submit">
        <div class="form-group">
          <label for="desk-name">Название колонки</label>
          <input type="text" id="desk-name" v-model="name" required />
        </div>
        <div class="buttons">
          <button type="submit" class="btn-save">Сохранить</button>
          <button type="button" class="btn-delete" @click="deleteDesk">Удалить</button>
          <button type="button" class="btn-cancel" @click="close">Отмена</button>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
import { thisUrl } from "@/utils/api";

export default {
  name: 'EditModal',
  props: {
    desk: Object
  },
  data() {
    return {
      name: this.desk.name,
    };
  },
  methods: {
    close() {
      this.$emit('close');
    },
    async submit() {
      try {
        const url = `${thisUrl()}/desks/edit/${this.desk.id}`;
        const userToken = localStorage.getItem('userToken');
        const response = await fetch(url, {
          method: 'PATCH',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${userToken}`
          },
          body: JSON.stringify({
            name: this.name,

          })
        });

        if (response.ok) {
          this.$emit('desk-updated');
          this.close();
        } else {
          console.error('Error updating desk:', response.statusText);
        }
      } catch (error) {
        console.error('Error updating desk:', error);
      }
    },
    async deleteDesk() {
      try {
        const url = `${thisUrl()}/desks/delete/${this.desk.id}`;
        const userToken = localStorage.getItem('userToken');
        const response = await fetch(url, {
          method: 'DELETE',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${userToken}`
          }
        });

        if (response.ok) {
          this.$emit('desk-updated');
          this.close();
        } else {
          console.error('Error deleting desk:', response.statusText);
        }
      } catch (error) {
        console.error('Error deleting desk:', error);
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
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background: #696969;
  padding: 20px;
  border-radius: 8px;
  width: 400px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.modal-content h2 {
  margin: 0 0 20px;
  color: #ECF0F1;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  color: #ECF0F1;
  margin-bottom: 5px;
}

.form-group input[type="text"],
.form-group input[type="color"] {
  width: 95%;
  padding: 8px;
  border: 1px solid #34495E;
  border-radius: 4px;
  background: #808080;
  color: #ECF0F1;
}

.buttons {
  display: flex;
  justify-content: space-between;
}

.buttons button {
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.btn-save {
  background-color: #27AE60;
  color: white;
}

.btn-save:hover {
  background-color: #229954;
}

.btn-delete {
  background-color: #E74C3C;
  color: white;
}

.btn-delete:hover {
  background-color: #C0392B;
}

.btn-cancel {
  background-color: #95A5A6;
  color: white;
}

.btn-cancel:hover {
  background-color: #7F8C8D;
}
</style>
