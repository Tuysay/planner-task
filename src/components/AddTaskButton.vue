<template>
  <button class="button-add">
    <span>+</span>
  </button>
</template>

<script>
export default {
  name: 'AddTaskButton'
};
</script>

<style scoped>
.button-add {
  position: relative; /* Изменено с relative на absolute */
  margin-top: 25%;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: #e74c3c;
  color: white;
  border: none;
  border-radius: 50%;
  width: 25px;
  height: 15px;
  font-size: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.2);
}
.button-add:hover {
  background-color: #c0392b;
}
</style>
