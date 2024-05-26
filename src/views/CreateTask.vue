<template>
  <div>
    <h1>Create Task</h1>
    <form>
      <div>
        <label for="title">Task Title:</label>
        <input type="text" id="title" />
      </div>
      <div>
        <label for="description">Task Description:</label>
        <textarea id="description"></textarea>
      </div>
      <button type="submit">Create Task</button>
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
