<template>
  <Loading :active="isLoading" />
  <Navbar />
  <div class="search">
    <Title />
    123
  </div>
</template>

<script>
import Navbar from '../components/Navbar.vue';
import Title from '../components/Title.vue';
import getAuthorizationHeader from '../tools/AuthorizationHeader';

export default {
  name: 'Search',
  components: {
    Navbar,
    Title,
  },
  data() {
    return {
      isLoading: false,
    };
  },
  methods: {
    getData(url, dataName) {
      this.isLoading = true;
      this.axios
        .get(url, {
          headers: getAuthorizationHeader(),
        })
        .then((response) => {
          this[dataName] = response.data;
        })
        .catch((err) => {
          console.log(err);
        });
      setTimeout(() => {
        this.isLoading = false;
      }, 1000);
    },
  },
  mounted() {},
};
</script>

<style lang="scss" scoped>
.search{
  width: calc(100% - 142px);
  padding: 55px 80px;
}
</style>
