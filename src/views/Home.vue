<template>
  <div>
    <nav class="navbar">
      <span v-if="!isAuthenticated">

        <router-link to="/login" class="navbar-item">Вход</router-link>
        <router-link to="/register" class="navbar-item">Регистрация</router-link>
      </span>
      <span v-else>
        <router-link to="/profile" class="navbar-item">Профиль</router-link>
        <router-link to="/calendar" class="navbar-item">Календарь</router-link>
        <router-link to="/" @click="logout" class="navbar-item">Выход</router-link>
        <span class="navbar-item">{{ userEmail }}</span>
      </span>
    </nav>
    <!-- Приветственное сообщение -->
    <div class="welcome-message" v-if="!isAuthenticated">
      <p>Добро пожаловать! Зарегистрируйтесь или войдите, чтобы начать использовать наше приложение.</p>
    </div>
    <div v-if="isAuthenticated" class="desks-container">
      <div v-if="desks.length === 0">
        <p class="poprob">Пока что у вас нет досок и задач, попробуйте добавить их!</p>
      </div>
      <div v-else class="desks-wrapper">
        <div v-for="desk in desks" :key="desk.id" class="desk-column">
          <h3>{{ desk.name }}</h3>
          <button @click="openEditModal(desk)" class="edit-button">Изменить</button>
        </div>
      </div>
    </div>
    <ButtonAdd @click="showModal = true" v-if="isAuthenticated" />
    <Modal v-if="showModal" @close="showModal = false" @task-created="fetchDesks" />
    <EditModal v-if="showEditModal" :desk="selectedDesk" @close="closeEditModal" @desk-updated="fetchDesks" />
  </div>
</template>

<script>
import ButtonAdd from '@/components/ButtonAdd.vue';
import Modal from '@/components/Modal.vue';
import EditModal from '@/components/EditModal.vue';
import { thisUrl } from '@/utils/api';

export default {
  components: {
    ButtonAdd,
    Modal,
    EditModal
  },
  data() {
    return {
      showModal: false,
      showEditModal: false,
      desks: [],
      selectedDesk: null
    };
  },
  computed: {
    isAuthenticated() {
      return !!localStorage.getItem('userToken');
    },
    userEmail() {
      return localStorage.getItem('userEmail');
    }
  },
  methods: {
    logout() {
      localStorage.removeItem('userToken');
      localStorage.removeItem('userEmail');
      this.$router.push('/');
      location.reload();
    },
    async fetchDesks() {
      try {
        const url = thisUrl() + "/desks";
        const userToken = localStorage.getItem('userToken');
        const response = await fetch(url, {
          method: 'GET',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${userToken}`
          }
        });
        if (response.ok) {
          const data = await response.json();
          console.log('Fetched desks:', data);
          this.desks = data.data.desks;
        } else {
          console.error('Error fetching desks:', response.statusText);
        }
      } catch (error) {
        console.error('Error fetching desks:', error);
      }
    },
    openEditModal(desk) {
      this.selectedDesk = desk;
      this.showEditModal = true;
    },
    closeEditModal() {
      this.selectedDesk = null;
      this.showEditModal = false;
    }
  },
  created() {
    if (this.isAuthenticated) {
      this.fetchDesks();
    }
  }
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
.poprob {
  color: orange;
  text-decoration: none;
  font-size: 36px;
  margin-top: 5%;
  margin-left: 5%;
}
.desks-container {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  padding: 20px;
}
.desks-wrapper {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  justify-content: center;
  width: 100%;
  max-width: 1200px;
}
.desk-column {
  background-color: #008B8B;
  border: 1px solid #2C3E50;
  border-radius: 8px;
  padding: 15px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  width: 80%;
  color: white;
  text-align: center;
  transition: transform 0.2s, box-shadow 0.2s;
  position: relative;
}
.desk-column:hover {
  transform: translateY(-5px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}
.desk-column h3 {
  margin: 0;
  font-size: 20px;
  color: #ECF0F1;
}
.edit-button {
  position: absolute;
  top: 10px;
  right: 10px;
  background-color: #2980B9;
  border: none;
  color: white;
  padding: 5px 10px;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s;
}
.edit-button:hover {
  background-color: #1F618D;
}

.welcome-message {
  color: orange;
  text-decoration: none;
  font-size: 36px;
  margin-top: 5%;
  margin-left: 5%;
}
</style>




<!--#008B8B-->