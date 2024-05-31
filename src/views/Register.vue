<template>
  <div>
    <nav class="navbar">
       <span v-if="!isAuthenticated">
        <router-link to="/" class="navbar-item">Главная</router-link>

        </span>
    </nav>
  </div>
  <div class="register">

    <h1>Регистрация</h1>
    <form @submit.prevent="register">
      <div class="form-group">
        <label for="name">Логин:</label>
        <input type="text" v-model="name" required>
      </div>
      <div class="form-group">
        <label for="email">Email:</label>
        <input type="email" v-model="email" placeholder="email" required>
      </div>
      <div class="form-group">
        <label for="password">Пароль:</label>
        <input type="password" v-model="password" placeholder="Password" minlength="6" required>
      </div>
      <button type="submit"  class="register-button">Зарегистрироваться</button>
      <button @click="toMain">Back</button>
    </form>
  </div>
</template>

<script>
import {thisUrl} from "@/utils/api";
export default {
  name: 'Register',
  data() {
    return {
      name: '',
      email: '',
      password: ''
    };
  },
  methods: {
    async register() {
      try {
        const url = thisUrl() + "/registration";
        const response = await fetch(url, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({
            name: this.name,
            email: this.email,
            password: this.password
          })
        });


      } catch (error) {
        console.error('Ошибка:', error);
        this.error = 'Ошибка регистрации';
        this.fio = ''
        this.email = ''
        this.password = ''
        this.errors = true;
        setTimeout(() => {
          this.errors = false;
        }, 3000);
      } finally {
        this.loading = false;
      }
    },
    toMain() {
      this.$router.push('/');
    }
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
}
.register-button:hover {
  background-color: #358a62;
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
