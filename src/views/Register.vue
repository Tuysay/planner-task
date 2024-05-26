<template>
  <div>
    <h1>Register Page</h1>
    <form>
      <div>
        <label for="email">Email:</label>
        <input type="email" id="email" />
      </div>
      <div>
        <label for="password">Password:</label>
        <input type="password" id="password" />
      </div>
      <div>
        <label for="confirm-password">Confirm Password:</label>
        <input type="password" id="confirm-password" />
      </div>
      <button type="submit">Register</button>
    </form>
  </div>
</template>

<style scoped>
form {
  display: flex;
  flex-direction: column;
}
div {
  margin-bottom: 10px;
}
button {
  background-color: #42b983;
  color: white;
  border: none;
  padding: 10px;
}
</style>
