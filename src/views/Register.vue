<template>
  <div>
    <nav class="navbar">
      <span v-if="!isAuthenticated">
        <router-link to="/" class="navbar-item">Главная</router-link>
      </span>
    </nav>
    <div class="container">
      <div class="animated-border">
        <div class="register">
          <h2>Регистрация</h2>
          <form @submit.prevent="register">
            <div class="inputBx">
              <input type="text" v-model="name" placeholder="Логин" minlength="4" maxlength="255" required>
              <div v-if="errors.name" class="error-message">{{ errors.name }}</div>
            </div>
            <div class="inputBx">
              <input type="email" v-model="email" placeholder="Email" required>
              <div v-if="errors.email" class="error-message">{{ errors.email }}</div>
            </div>
            <div class="inputBx">
              <input type="password" v-model="password" placeholder="Password" minlength="8" maxlength="25" required>
              <div v-if="errors.password" class="error-message">{{ errors.password }}</div>
            </div>
            <div class="inputBx">
              <input type="file" @change="onFileChange">
              <div v-if="errors.avatar" class="error-message">{{ errors.avatar }}</div>
            </div>
            <div class="inputBx">
              <input type="submit" value="Зарегистрироваться" class="register-button" :disabled="loading">
            </div>
            <div v-if="loading" class="loader"></div>
            <div v-if="error" class="error-message">{{ error }}</div>
          </form>
        </div>
      </div>
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
      errors: {},
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
          body: formData,
        });

        if (response.ok) {
          this.name = '';
          this.email = '';
          this.password = '';
          this.avatar = null;
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
  }
};
</script>

<style scoped>


* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Open Sans', sans-serif;
}

body {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background: #111;
}

.container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
}

.animated-border {
  position: relative;
  width: 500px;
  height: 500px;
  display: flex;
  justify-content: center;
  align-items: center;
}



.register {
  position: relative;
  width: 300px;
  background: rgba(0, 0, 0, 0.5);
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
  text-align: center;
}

.register h2 {
  font-size: 2em;
  color: #fff;
  margin-bottom: 20px;
}

.inputBx {
  position: relative;
  width: 100%;
  margin-bottom: 15px; /* Отступ между полями */
}

.inputBx input {
  position: relative;
  width: 100%;
  padding: 12px 20px;
  background: transparent;
  border: 2px solid #fff;
  border-radius: 40px;
  font-size: 1.2em;
  color: #fff;
  box-shadow: none;
  outline: none;
}

.inputBx input::placeholder {
  color: rgba(255, 255, 255, 0.75);
}

.inputBx input[type="submit"] {
  background: linear-gradient(45deg, #3549ff, #72deff);
  border: none;
  cursor: pointer;
  font-size: 1.2em;
  padding: 10px 20px;
  color: white;
  border-radius: 5px;
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
