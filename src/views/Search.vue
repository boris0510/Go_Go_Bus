<template>
  <Loading :active="isLoading" />
  <Navbar />
</template>

<script>
import Navbar from '../components/Navbar.vue';
import getAuthorizationHeader from '../tools/AuthorizationHeader';

export default {
  name: 'Search',
  components: {
    Navbar,
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
