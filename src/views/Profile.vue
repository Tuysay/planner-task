<template>
  <div class="profile">
    <h1>Профиль</h1>
    <div class="profile-info">
      <img :src="user.avatar" alt="User Avatar" class="avatar">
      <div class="details">
        <p><strong>Логин:</strong> {{ user.username }}</p>
        <p><strong>О себе:</strong></p>
        <textarea v-model="user.bio"></textarea>
        <button class="save-button" @click="saveProfile">Сохранить</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Profile',
  data() {
    return {
      user: {
        username: 'exampleUser',
        avatar: 'https://via.placeholder.com/150',
        bio: ''
      }
    };
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
</style>
