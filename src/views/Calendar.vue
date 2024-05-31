<template>
  <div class="calendar">
    <div>
      <nav class="navbar">
       <span v-if="!isAuthenticated">
        <router-link to="/login" class="navbar-item">Вход</router-link>
        <router-link to="/register" class="navbar-item">Регистрация</router-link>
        </span>
        <span v-if="isAuthenticated">
      <router-link to="/profile" class="navbar-item">Профиль</router-link>
      <router-link to="/calendar" class="navbar-item">Календарь</router-link>
        <router-link to="/" @click="logout">Sign out</router-link>
        </span>
      </nav>
    </div>
    <h1>Calendar Page</h1>
  </div>
</template>

<script>
export default {
  name: 'Calendar',
  methods: {
    logout() {
      localStorage.removeItem('userToken');
      this.$router.push('/');
      location.reload();
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
.calendar {
  padding: 20px;
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
