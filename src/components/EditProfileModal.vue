<template>
  <div class="modal-overlay" @click.self="close">
    <div class="modal-content">
      <h2>Изменить профиль</h2>
      <form @submit.prevent="submit">
        <div class="form-group">
          <label for="user-name">Логин</label>
          <input type="text" id="user-name" v-model="name" required />
        </div>
        <div class="buttons">
          <button type="submit" class="btn-save">Сохранить</button>
          <button type="button" class="btn-cancel" @click="close">Отмена</button>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
import { thisUrl } from "@/utils/api";

export default {
  name: 'EditProfileModal',
  props: {
    user: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      name: this.user.name,
    };
  },
  methods: {
    close() {
      this.$emit('close');
    },
    async submit() {
      try {
        const userToken = localStorage.getItem('userToken');
        const response = await fetch(`${thisUrl()}/users/edit/${this.user.id}`, {
          method: 'PATCH',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${userToken}`
          },
          body: JSON.stringify({
            name: this.name
          })
        });

        if (response.ok) {
          this.$emit('profile-updated');
          this.close();
        } else {
          console.error('Error updating profile:', response.statusText);
        }
      } catch (error) {
        console.error('Error updating profile:', error);
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
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 5px;
  width: 90%;
  max-width: 500px;
}

.form-group {
  margin-bottom: 15px;
}

label {
  display: block;
  margin-bottom: 5px;
}

input[type="text"] {
  width: 100%;
  padding: 10px;
  font-size: 16px;
}

.buttons {
  display: flex;
  justify-content: space-between;
}

.btn-save,
.btn-cancel {
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.btn-save {
  background-color: #42b983;
  color: white;
}

.btn-cancel {
  background-color: #ccc;
  color: black;
}

.btn-save:hover {
  background-color: #358a62;
}

.btn-cancel:hover {
  background-color: #999;
}
</style>
