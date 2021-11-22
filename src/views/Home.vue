<template>
  <div class="home">
    <h1>Go Go Bus</h1>
  </div>
</template>

<script>
export default {
  name: 'Home',
  mounted() {
    // setTimeout(() => {
    //   this.$router.push('/nearby');
    // }, 2500);
  },
};
</script>

<style lang="scss" scoped>
.home {
  width: 100%;
  height: 100vh;
  background-image: url('../assets/img/index.svg');
  background-size: cover;
  background-position: center center;
  h1 {
    text-indent: 101%;
    overflow: hidden;
    white-space: nowrap;
  }
}
@media (max-width: 1024px) {
  .home {
    background-image: url('../assets/img/index_mb.svg');
  }
}
</style>
