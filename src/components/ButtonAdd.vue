<template>
  <button class="button-add">
    <span>+</span>
  </button>
</template>

<script>
export default {
  name: 'ButtonAdd'
};
</script>

<style scoped>
.button-add {
  position: absolute; /* Изменено с relative на absolute */
  margin-bottom: clamp(10%, 20%, 30%);
  margin-top: 25%;
  margin-left: 3%;
  transform: translate(-50%, -50%);
  background-color: #e74c3c;
  color: white;
  border: none;
  border-radius: 50%;
  width: 60px;
  height: 60px;
  font-size: 30px;
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
