<template>
  <div>
    <nav class="navbar">
      <span v-if="!isAuthenticated">
        <router-link to="/" class="navbar-item">Главная</router-link>
      </span>
    </nav>
    <div class="login">
      <h1>Вход</h1>
      <form @submit.prevent="login">
        <div class="form-group">
          <label for="email">Email:</label>
          <input type="email" v-model="email" placeholder="email" required>
        </div>
        <div class="form-group">
          <label for="password">Пароль:</label>
          <input type="password" id="password" v-model="password" minlength="8" required>
        </div>
        <button type="submit" class="login-button" :disabled="loading">
          <span v-if="loading" class="loader"></span>
          <span v-else>Войти</span>
        </button>
        <div v-if="error" class="error-message">{{ error }}</div>
      </form>
    </div>
  </div>
</template>

<script>
import { thisUrl } from "@/utils/api";

export default {
  data() {
    return {
      email: '',
      password: '',
      loading: false,
      error: null,
    };
  },
  methods: {
    async login() {
      this.loading = true;
      this.error = null;

      try {
        const url = thisUrl() + "/login";
        const response = await fetch(url, {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            email: this.email,
            password: this.password,
          }),
        });

        if (response.ok) {
          const userToken = await response.json();
          localStorage.setItem("userToken", userToken.data.user_token);
          this.$router.push("/");
        } else {
          if (response.status === 401) {
            this.error = "Неправильный логин или пароль";
          } else if (response.status === 409) {
            this.error = "Пользователь с таким email уже существует";
          } else {
            this.error = "Ошибка входа";
          }
          this.resetForm();
        }
      } catch (error) {
        console.error("Ошибка:", error);
        this.error = "Ошибка входа";
        this.resetForm();
      } finally {
        this.loading = false;
      }
    },
    resetForm() {
      this.email = "";
      this.password = "";
      setTimeout(() => {
        this.error = null;
      }, 3000);
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
.login {
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
.login-button {
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
.login-button:disabled {
  background-color: #a5d6a7;
  cursor: not-allowed;
}
.login-button:hover {
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
