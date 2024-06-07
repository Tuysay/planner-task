<template>
  <div>
    <nav class="navbar">
      <span v-if="!isAuthenticated">
        <router-link to="/" class="navbar-item">Главная</router-link>
      </span>
    </nav>
    <div class="register">
      <h1>Регистрация</h1>
      <form @submit.prevent="register">
        <div class="form-group">
          <label for="name">Логин:</label>
          <input type="text" v-model="name" minlength="4" required>
          <div v-if="errors.name" class="error-message">{{ errors.name }}</div>
        </div>
        <div class="form-group">
          <label for="email">Email:</label>
          <input type="email" v-model="email" placeholder="email" required>
          <div v-if="errors.email" class="error-message">{{ errors.email }}</div>
        </div>
        <div class="form-group">
          <label for="password">Пароль:</label>
          <input type="password" v-model="password" placeholder="Password" minlength="8" required>
          <div v-if="errors.password" class="error-message">{{ errors.password }}</div>
        </div>
        <div class="form-group">
          <label for="avatar">Фото профиля:</label>
          <input type="file" @change="onFileChange">
          <div v-if="errors.avatar" class="error-message">{{ errors.avatar }}</div>
        </div>
        <button type="submit" class="register-button" :disabled="loading">
          <span v-if="loading" class="loader"></span>
          <span v-else>Зарегистрироваться</span>
        </button>
        <div v-if="error" class="error-message">{{ error }}</div>
      </form>
    </div>
  </div>
</template>

<script>
import { thisUrl } from "@/utils/api";

export default {
  name: 'Register',
  data() {
    return {
      name: '',
      email: '',
      password: '',
      avatar: null,
      loading: false,
      error: null,
      errors: {}, // Объект для хранения ошибок валидации
    };
  },
  methods: {
    onFileChange(e) {
      this.avatar = e.target.files[0];
    },
    async register() {
      this.loading = true;
      this.error = null;
      this.errors = {};

      try {
        const url = thisUrl() + "/registration";
        const formData = new FormData();
        formData.append('name', this.name);
        formData.append('email', this.email);
        formData.append('password', this.password);
        formData.append('avatar', this.avatar || '');

        const response = await fetch(url, {
          method: "POST",
          headers: {
            "Accept": "application/json",
          },
          body: formData,
        });

        if (response.ok) {
          this.$router.push("/login");
        } else {
          const data = await response.json();
          if (response.status === 409) {
            this.errors.email = "Такая почта уже была зарегистрирована";
          } else if (response.status === 422 && data.errors) {
            this.errors = data.errors; // Получаем ошибки валидации с сервера
          } else {
            this.error = "Ошибка регистрации";
          }
        }
      } catch (error) {
        console.error("Ошибка:", error);
        this.error = "Ошибка регистрации";
      } finally {
        this.loading = false;
      }
    },
  },
  computed: {
    isAuthenticated() {
      return !!localStorage.getItem('userToken');
    }
  },
};
</script>

<style scoped>
.register {
  padding: 20px;
  max-width: 400px;
  margin: 0 auto;
}
.form-group {
  margin-bottom: 15px;
}
label {
  display: block;
  margin-bottom: 5px;
}
input {
  width: 100%;
  padding: 10px;
  font-size: 16px;
}
.register-button {
  padding: 10px 20px;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}
.register-button:disabled {
  background-color: #a5d6a7;
  cursor: not-allowed;
}
.register-button:hover {
  background-color: #358a62;
}
.error-message {
  color: red;
  margin-top: 10px;
}
.loader {
  border: 4px solid #f3f3f3;
  border-radius: 50%;
  border-top: 4px solid #3498db;
  width: 20px;
  height: 20px;
  -webkit-animation: spin 2s linear infinite; /* Safari */
  animation: spin 2s linear infinite;
}
@-webkit-keyframes spin {
  0% { -webkit-transform: rotate(0deg); }
  100% { -webkit-transform: rotate(360deg); }
}
@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
.navbar {
  display: flex;
  justify-content: center;
  background-color: #2C3E50;
  padding: 10px;
}
.navbar-item {
  color: orange;
  margin: 0 10px;
  text-decoration: none;
}
.navbar-item:hover {
  text-decoration: underline;
}
</style>
