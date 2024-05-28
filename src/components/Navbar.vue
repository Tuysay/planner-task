<template>
  <nav class="navbar">
    <router-link to="/login" class="navbar-item">Вход</router-link>
    <router-link to="/register" class="navbar-item">Регистрация</router-link>
    <router-link to="/profile" class="navbar-item">Профиль</router-link>
    <router-link to="/calendar" class="navbar-item">Календарь</router-link>
  </nav>
</template>

<script>
export default {
  name: 'Navbar'
};
</script>

<style scoped>
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
