<template>
  <div>
    <nav class="navbar">
      <span v-if="!isAuthenticated">
        <router-link to="/login" class="navbar-item">Вход</router-link>
        <router-link to="/register" class="navbar-item">Регистрация</router-link>
      </span>
      <span v-if="isAuthenticated">
        <router-link to="/" class="navbar-item">Главная</router-link>
        <router-link to="/calendar" class="navbar-item">Календарь</router-link>
        <router-link to="/" @click="logout">Sign out</router-link>
      </span>
    </nav>
    <div class="profile">
      <h1>Профиль</h1>
      <div class="profile-info">
<!--        <img :src="users.avatar" alt="User Avatar" class="avatar">-->
        <div class="details">
          <p>Логин: {{ users.name }}</p>
          <p>Дата создания: {{ users.created_at }}</p>
          <button @click="openEditModal" class="edit-button">Изменить</button>
        </div>
      </div>
      <EditProfileModal
          v-if="isEditModalOpen"
          :user="users"
          @close="isEditModalOpen = false"
          @profile-updated="fetchUserProfile"
      />
    </div>
  </div>
</template>

<script>
import EditProfileModal from '@/components/EditProfileModal.vue';
import { thisUrl } from "@/utils/api";

export default {
  name: 'Profile',
  components: {
    EditProfileModal
  },
  data() {
    return {
      users: {
        id: '',
        name: '',
        avatar: '',
        created_at: ''
      },
      isEditModalOpen: false
    };
  },
  mounted() {
    this.fetchUserProfile();
  },
  methods: {
    logout() {
      localStorage.removeItem('userToken');
      this.$router.push('/');
      location.reload();
    },
    async fetchUserProfile() {
      try {
        const url = thisUrl() + "/user";
        const userToken = localStorage.getItem('userToken');
        const response = await fetch(url, {
          method: 'GET',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${userToken}`
          }
        });
        if (response.ok) {
          const userData = await response.json();
          console.log('Fetched user profile:', userData);
          this.users = userData;
        } else {
          console.error('Error fetching user profile:', response.statusText);
        }
      } catch (error) {
        console.error('Error fetching user profile:', error);
      }
    },
    openEditModal() {
      this.isEditModalOpen = true;
    }
  },
  computed: {
    isAuthenticated() {
      return !!localStorage.getItem('userToken');
    }
  }
};
</script>

<style scoped>
.profile {
  padding: 20px;
  max-width: 600px;
  margin: 0 auto;
}

.profile-info {
  display: flex;
  align-items: flex-start;
}

.avatar {
  width: 150px;
  height: 150px;
  border-radius: 50%;
  margin-right: 20px;
}

.details {
  flex: 1;
}

.edit-button {
  padding: 10px 20px;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.edit-button:hover {
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
