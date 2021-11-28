<template>
  <div class="navbar">
    <div class="navbar-logo">
      <a href="#" to="/"><img src="../assets/img/logo.png" alt="logo" /></a>
    </div>
    <div class="navbar-link">
      <router-link to="/nearby"><img src="../assets/img/icon_nearby.png" alt="" /></router-link>
      <router-link to="/search"><img src="../assets/img/icon_search.png" alt="" /></router-link>
      <router-link to="/news"><img src="../assets/img/icon_info.png" alt="" /></router-link>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.navbar {
  width: 142px;
  height: 100vh;
  background: #486ae8;
  box-shadow: 3px 2px 6px rgba(121, 121, 121, 0.25);
  border-radius: 0px 16px 16px 0px;
  .navbar-logo {
    display: flex;
    justify-content: center;
    align-items: center;
    padding-top: 40px;
    padding-bottom: 45px;
    border-bottom: 1px solid rgba(255, 255, 255, 0.5);
  }
  .navbar-link {
    display: flex;
    flex-direction: column;
    align-items: center;
    a:first-child {
      margin-top: 27px;
    }
    a {
      width: 63px;
      height: 63px;
      display: flex;
      justify-content: center;
      align-items: center;
      margin-bottom: 40px;
      border-radius: 16px;
      transition: all 0.5s;
      &:hover {
        background: #2a4ac2;
      }
      &.active {
        background: #2a4ac2;
      }
    }
  }
}
</style>
