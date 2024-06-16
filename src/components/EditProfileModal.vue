<!--окно изменения профиля-->
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
@import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@400;600&display=swap');

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.75); /* Darker background */
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px; /* Add some padding for smaller screens */
}

.modal-content {
  background: #696969; /* Dark background for modal */
  padding: 30px;
  border-radius: 10px; /* Increased border radius */
  width: 100%;
  max-width: 400px; /* Reduced max width */
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2); /* Softer shadow */
  color: #ffffff; /* White text color */
  font-family: 'Montserrat', sans-serif; /* Changed font */
}

h2 {
  font-size: 1.8em; /* Slightly smaller font size */
  font-weight: 600;
  margin-bottom: 20px; /* More space below heading */
}

.form-group {
  margin-bottom: 20px; /* Increased margin between form groups */
}

label {
  display: block;
  font-weight: 600; /* Bold labels */
  margin-bottom: 8px; /* Increased margin below labels */
}

input[type="text"] {
  width: 95%;
  padding: 12px; /* Increased padding */
  font-size: 1em; /* Standardized font size */
  border: 2px solid #ccc;
  border-radius: 5px; /* Standard border radius */
  background: #808080; /* Dark background for input */
  color: #ffffff; /* White text color for input */
}

input[type="text"]::placeholder {
  color: #aaaaaa; /* Light grey placeholder text */
}

input[type="text"]:focus {
  border-color: #42b983; /* Highlighted border color */
  outline: none; /* Remove outline */
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
  transition: background-color 0.3s ease; /* Smooth background transition */
  font-weight: 600; /* Bold text */
}

.btn-save {
  background-color: #42b983;
  color: white;
}

.btn-cancel {
  background-color: #d9534f; /* Red color for cancel button */
  color: white;
}

.btn-save:hover {
  background-color: #358a62;
}

.btn-cancel:hover {
  background-color: #c9302c; /* Darker red on hover */
}
</style>
