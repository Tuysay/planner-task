<template>
  <div id="app" :style="appStyle">
    <router-view />
    <button @click="toggleTheme" class="theme-button">Сменить стиль</button>
  </div>
</template>

<script>

export default {
  components: {
  },
  data() {
    return {
      darkTheme: true
    };
  },
  computed: {
    appStyle() {
      return {
        backgroundColor: this.darkTheme ? '#34495E' : '#D3D3D3', // Фоновый цвет в зависимости от темы
        color: this.darkTheme ? '#D3D3D3' : '#34495E' // Цвет текста в зависимости от темы
      };
    }
  },
  mounted() {
    // При загрузке страницы читаем значение из localStorage и устанавливаем его
    const savedTheme = localStorage.getItem('darkTheme');
    if (savedTheme !== null) {
      this.darkTheme = JSON.parse(savedTheme);
      if (this.darkTheme) {
        document.body.style.backgroundColor = '#34495E'; // Изменяем фон body при переключении темы
      }
    }
  },
  methods: {
    toggleTheme() {
      this.darkTheme = !this.darkTheme; // Переключаем состояние темы
      if (this.darkTheme) {
        document.body.style.backgroundColor = '#34495E'; // Изменяем фон body при переключении темы
      } else {
        document.body.style.backgroundColor = '#D3D3D3';
      }
      // Сохраняем значение в localStorage при изменении темы
      localStorage.setItem('darkTheme', JSON.stringify(this.darkTheme));
    }
  }
}
</script>

<style>
body {
  margin: 0;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  background-color: #D3D3D3; /* Устанавливаем фон body по умолчанию */
}
nav {
  margin-bottom: 20px;
}
nav a {
  margin: 0 10px;
  text-decoration: none;
  color: #42b983;
}
nav a.router-link-exact-active {
  font-weight: bold;
  color: #35495e;
}
.theme-button {
  position: fixed; /* Позиционируем кнопку */
  bottom: 20px; /* Располагаем её внизу экрана */
  right: 20px; /* и справа */
  padding: 10px 20px;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
.theme-button:hover {
  background-color: #358a62;
}
</style>
