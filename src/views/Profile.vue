<template>
  <div class="profile">
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
    </div>
    <h1>Профиль</h1>
    <div class="profile-info">
      <img :src="users.avatar" alt="User Avatar" class="avatar">
      <div class="details">
        <label for="avatar-upload" class="avatar-upload-label">Изменить фото</label>
        <input type="file" id="avatar-upload" @change="handleAvatarUpload" accept="image/*" class="avatar-upload">
        <p>Логин {{ users.name }}</p>

      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Profile',
  data() {
    return {
      users: {
        name: '',
        avatar: 'avatars/',
      },
      newAvatar: null
    };
  },
  methods: {
    logout() {
      localStorage.removeItem('userToken');
      this.$router.push('/');
      location.reload();
    },
    handleAvatarUpload(event) {
      const file = event.target.files[0];
      if (file && file.type.startsWith('image/')) {
        this.newAvatar = URL.createObjectURL(file);
        this.user.avatar = this.newAvatar;
      }
    },
    saveProfile() {
      // Implement the logic to save the profile here, including the new avatar if applicable.
      console.log('Profile saved:', this.users);
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
textarea {
  width: 100%;
  min-height: 100px;
  padding: 10px;
  font-size: 16px;
}
input[type="text"] {
  width: 100%;
  padding: 10px;
  font-size: 16px;
  margin-bottom: 15px;
}
.avatar-upload-label {
  display: inline-block;
  margin-bottom: 10px;
  cursor: pointer;
  color: #42b983;
}
.avatar-upload {
  display: none;
}
.save-button {
  padding: 10px 20px;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
.save-button:hover {
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
