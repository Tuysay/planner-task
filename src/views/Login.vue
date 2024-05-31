
<template>
  <div>
    <nav class="navbar">
       <span v-if="!isAuthenticated">
        <router-link to="/" class="navbar-item">Главная</router-link>

        </span>
    </nav>
  </div>
  <div class="login">

    <h1>Вход</h1>
    <form @submit.prevent="login">
      <div class="form-group">
        <label for="email">Email:</label>
        <input type="email" v-model="email" placeholder="email" required>
      </div>
      <div class="form-group">
        <label for="password">Пароль:</label>
        <input type="password" id="password" v-model="password" required>
      </div>
      <button type="submit">Login</button>
    </form>
  </div>
</template>

<script>
import { thisUrl } from "@/utils/api";

export default {
  data() {
    return {
      email: '',
      password: "",

    };
  },
  methods: {
    async login() {
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
          } else {
            this.error = "Ошибка входа";
          }
          this.email = "";
          this.password = "";
          this.errors = true;
          setTimeout(() => {
            this.errors = false;
          }, 3000);
        }
      } catch (error) {
        console.error("Ошибка:", error);
        this.error = "Ошибка входа";
        this.email = "";
        this.password = "";
        this.errors = true;
        setTimeout(() => {
          this.errors = false;
        }, 3000);
      }

    },
    computed: {
      isAuthenticated() {
        return !!localStorage.getItem('userToken');
      }
    },
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
}
.login-button:hover {
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
