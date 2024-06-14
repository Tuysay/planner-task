<template>
  <div class="profile-container">
    <nav class="navbar">
      <span v-if="!isAuthenticated">
        <router-link to="/login" class="navbar-item">Вход</router-link>
        <router-link to="/register" class="navbar-item">Регистрация</router-link>
      </span>
      <span v-else>
        <router-link to="/" class="navbar-item">Главная</router-link>
        <router-link to="/profile" class="navbar-item">Профиль</router-link>
        <router-link to="/" @click="logout" class="navbar-item">Выход</router-link>
      </span>
    </nav>
    <div class="profile-content">
      <h1>Профиль</h1>
      <div v-if="isAuthenticated" class="profile-info">
        <div class="avatar-wrapper">
          <img :src="users.avatar || '/avatars/kj8DXcMNKxKdXLWAA8ZX5gRzo953pjvh.svg'" alt="User Avatar" class="avatar">
        </div>
        <div class="details">
          <table>
            <tr>
              <td><strong>Логин:</strong></td>
              <td>{{ users.name }}</td>
            </tr>
            <tr>
              <td><strong>Дата создания:</strong></td>
              <td>{{ formatDate(users.created_at) }}</td>
            </tr>
          </table>
          <button @click="openEditModal" class="edit-button">Редактировать профиль</button>
          <button @click="deleteAccount" class="delete-button">Удалить аккаунт</button>
        </div>
      </div>
    </div>
    <EditProfileModal
        v-if="isEditModalOpen"
        :user="users"
        @close="isEditModalOpen = false"
        @profile-updated="fetchUserProfile"
    />
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
    },
    formatDate(dateString) {
      const date = new Date(dateString);
      return `${date.getDate()}.${date.getMonth() + 1}.${date.getFullYear()}`;
    },
    async deleteAccount() {
      if (!confirm("Вы уверены, что хотите удалить свой аккаунт? Это действие нельзя будет отменить.")) {
        return;
      }

      try {
        const url = thisUrl() + `/users/delete/${this.users.id}`;
        const userToken = localStorage.getItem('userToken');
        const response = await fetch(url, {
          method: 'DELETE',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${userToken}`
          }
        });

        if (response.ok) {
          this.logout();
        } else {
          const errorData = await response.json();
          console.error('Error deleting account:', errorData);
        }
      } catch (error) {
        console.error('Error deleting account:', error);
      }
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
.profile-container {
  max-width: 800px;
  margin: 0 auto;
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

.profile-content {
  margin-top: 20px;
  background-color: #f2f2f2;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.profile-info {
  display: flex;
  align-items: center;
}

.avatar-wrapper {
  width: 150px;
  height: 150px;
  overflow: hidden;
  border-radius: 50%;
  margin-right: 20px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.avatar {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.details {
  flex: 1;
}

.details table {
  width: 100%;
  margin-top: 10px;
}

.details table td {
  padding: 8px 0;
  color: black;
}

.edit-button {
  margin-top: 10px;
  padding: 10px 20px;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s, box-shadow 0.3s;
}

.edit-button:hover {
  background-color: #358a62;
}

.delete-button {
  margin-top: 10px;
  padding: 10px 20px;
  background-color: #e74c3c;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s, box-shadow 0.3s;
  margin-left: 40%;
}

.delete-button:hover {
  background-color: #c0392b;
}
</style>
