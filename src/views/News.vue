<template>
  <Loading :active="isLoading" />
  <Navbar />
  <div class="news">
    <Title />
    <div class="news-content">
      <div class="news-content-title">
        <h2>最新消息</h2>
        <select v-model="currentCategory" @change="getNews">
          <option value="">請選擇縣市</option>
          <option :value="item.City" v-for="item in city" :key="item">
            {{ item.CityName }}
          </option>
        </select>
      </div>
      <div class="information">
        <div class="information-content">
          <template v-if="news.length !== 0">
            <div class="information-txt" v-for="item in news" :key="item.NewsID">
              <p>{{ item.PublishTime.replace('T', ' ').replace('+08:00', '') }}</p>
              <h3>{{ item.Description }}</h3>
            </div>
          </template>
          <div class="information-txt" v-if="news.length === 0">
            <p></p>
            <h3 style="color: #486ae8; font-weight: bold">請選擇縣市 !</h3>
          </div>
        </div>
      </div>
    </div>
    <Footer />
  </div>
</template>

<script>
import Navbar from '../components/Navbar.vue';
import Title from '../components/Title.vue';
import Footer from '../components/Footer.vue';
import getAuthorizationHeader from '../tools/AuthorizationHeader';

export default {
  name: 'News',
  components: {
    Navbar,
    Title,
    Footer,
  },
  data() {
    return {
      city: [],
      currentCategory: '',
      news: [],
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
    getNews() {
      this.getData(
        `https://ptx.transportdata.tw/MOTC/v2/Bus/News/City/${this.currentCategory}?$format=JSON`,
        'news',
      );
    },
  },
  mounted() {
    this.getData('https://gist.motc.gov.tw/gist_api/V3/Map/Basic/City?$format=JSON', 'city');
  },
};
</script>

<style lang="scss" scoped>
.news {
  width: calc(100% - 142px);
  padding: 55px 80px 0 80px;
  .news-content {
    background: #ffffff;
    box-shadow: 2px 4px 7px rgba(196, 196, 196, 0.11);
    border-radius: 16px;
    margin-top: 40px;
    .news-content-title {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding-top: 50px;
      padding-left: 70px;
      padding-right: 70px;
      h2 {
        font-weight: bold;
        font-size: 20px;
        color: #7d7d7d;
      }
      select {
        font-weight: bold;
        font-size: 20px;
        color: #7d7d7d;
      }
    }
    .information {
      padding-left: 70px;
      padding-right: 70px;
      padding-bottom: 50px;
      margin-top: 35px;
      .information-content {
        height: calc(100vh - 355px);
        overflow-y: scroll;
        border: 1px solid #f3f3f3;
        .information-txt {
          padding: 25px 0;
          padding-left: 70px;
          background: #f6f7ff;
          margin-top: 3px;
          &:first-child {
            margin-top: 0;
          }
          &:nth-child(2n) {
            background: #eceeff;
          }
          h3 {
            font-weight: 500;
            font-size: 18px;
            color: #7d7d7d;
            margin-top: 5px;
          }
          p {
            font-weight: bold;
            font-size: 18px;
            color: #486ae8;
          }
        }
      }
    }
  }
}
</style>
