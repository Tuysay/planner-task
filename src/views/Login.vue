<template>
  <div>
    <nav class="navbar">
      <span v-if="!isAuthenticated">
        <router-link to="/" class="navbar-item">Главная</router-link>
        <router-link to="/register" class="navbar-item">Регистрация</router-link>
      </span>
    </nav>
    <div class="container">
      <div class="cont1">
        <i style="--clr:#ffffff;"></i>
        <i style="--clr:#3331ac;"></i>
        <i style="--clr:#a52d2d;"></i>
        <div class="login">
          <h2>Вход</h2>
          <form @submit.prevent="login">
            <div class="inputBx">
              <input type="email" v-model="email" placeholder="Email" required>
            </div>
            <div class="inputBx">
              <input type="password" id="password" v-model="password" placeholder="Пароль" minlength="8" required>
            </div>
            <div class="inputBx">
              <input type="submit" value="Войти" class="login-button" :disabled="loading">
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
          localStorage.setItem("userToken", userToken.data.token);
          localStorage.setItem("userEmail", this.email); // Сохранение email в localStorage
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
@import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@300&display=swap');
@import url('https://fonts.googleapis.com/css?family=Open+Sans:400,600,700&display=swap');

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

.cont1 {
  position: relative;
  width: 500px;
  height: 500px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.cont1 i {
  position: absolute;
  inset: 0;
  border: 2px solid #fff;
  transition: 0.5s;
  border: 10px double white;
}

.cont1 i:nth-child(1) {
  border-radius: 0% 100% 0% 100% / 100% 0% 100% 0%;
  animation: animate 6s linear infinite;
}

.cont1 i:nth-child(2) {
  border-radius: 0% 100% 0% 100% / 100% 0% 100% 0%;
  animation: animate 4s linear infinite;
}

cont1 i:nth-child(3) {
  border-radius: 0% 100% 0% 100% / 100% 0% 100% 0%;
  animation: animate2 10s linear infinite;
}

@keyframes animate {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

@keyframes animate2 {
  0% {
    transform: rotate(360deg);
  }
  100% {
    transform: rotate(0deg);
  }
}

.cont1:hover i {
  border: 20px dashed var(--clr);
  filter: drop-shadow(0 0 20px var(--clr));
}

.login {
  position: absolute;
  width: 300px;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  gap: 20px;
  color: aqua;
}

.login h2 {
  font-size: 2em;
  color: #fff;
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
