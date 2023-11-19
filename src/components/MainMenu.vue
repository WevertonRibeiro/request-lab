<template>
  <ul class="main_menu">
    <li class="logo">
      <img :src="logo" />
    </li>
    <!-- <li>
      <img :src="logo" />
    </li> -->
  </ul>
</template>

<script setup>
import logo from '../assets/fire-logo.png'
</script>

<style lang="scss" scoped>
.main_menu {
  display: flex;
  flex-direction: column;
  align-items: center;
  list-style: none;
  padding: 0;
  margin: 0;
  // background: pink;

  li {
    display: flex;
    align-items: center;
    justify-content: center;
    img {
      width: 24px;
      height: 24px;
    }
    &.logo {
      height: 70px;
      margin-bottom: 15px;
      img {
        border-radius: 50%;
      }
    }
  }
}
</style>