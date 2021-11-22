<template>
  <Loading :active="isLoading" />
  <Navber />
  <div class="nearby">
    <div class="nearby-head">
      <h1>BUS STOP | 附近站牌</h1>
    </div>
    <div class="nearby-content">
      <h2>附近的站牌</h2>
      <div class="distance">
        <div class="distance-title">
          <div class="distance-common distance-100 active" @click="getNearbyBusSite(500)">
            500 m
          </div>
          <div class="distance-common distance-300" @click="getNearbyBusSite(1000)">1000 m</div>
        </div>
        <div class="distance-content">
          <div class="distance-txt" v-for="item in nearbyBusSite" :key="item.StationID">
            <h3>
              {{ item.StationName.Zh_tw }}
              <span v-if="item.Bearing === 'E'">(東行)</span>
              <span v-if="item.Bearing === 'W'">(西行)</span>
              <span v-if="item.Bearing === 'S'">(南行)</span>
              <span v-if="item.Bearing === 'N'">(北行)</span>
              <span v-if="item.Bearing === 'SE'">(東南行)</span>
              <span v-if="item.Bearing === 'NE'">(東北行)</span>
              <span v-if="item.Bearing === 'SW'">(西南行)</span>
              <span v-if="item.Bearing === 'NW'">(西北行)</span>
            </h3>
            <p v-for="item2 in item.Stops" :key="item2">
              {{ item2.RouteName.Zh_tw }}
            </p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Navber from '../components/Navber.vue';
import getAuthorizationHeader from '../tools/AuthorizationHeader';

export default {
  name: 'Nearby',
  components: {
    Navber,
  },
  data() {
    return {
      userLocation: {
        longitude: 0,
        latitude: 0,
      },
      nearbyBusSite: [],
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
    getUserLocation() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          (position) => {
            this.userLocation.longitude = position.coords.longitude;
            this.userLocation.latitude = position.coords.latitude;
            this.getNearbyBusSite(500);
          },
          (e) => {
            const msg = e.message;
            console.error(msg);
          },
        );
      }
    },
    getNearbyBusSite(m) {
      this.getData(
        `https://ptx.transportdata.tw/MOTC/v2/Bus/Station/NearBy?$spatialFilter=nearby(${this.userLocation.latitude}%2C%20${this.userLocation.longitude}%2C%20${m})&$format=JSON`,
        'nearbyBusSite',
      );
    },
  },
  mounted() {
    this.getUserLocation();

    const distance100 = document.querySelector('.distance-100');
    const distance300 = document.querySelector('.distance-300');
    distance100.addEventListener('click', () => {
      distance100.classList.add('active');
      distance300.classList.remove('active');
    });
    distance300.addEventListener('click', () => {
      distance100.classList.remove('active');
      distance300.classList.add('active');
    });
  },
};
</script>

<style lang="scss" scoped>
.nearby {
  width: calc(100% - 142px);
  padding: 55px 80px;
  .nearby-head {
    h1 {
      font-weight: bold;
      font-size: 26px;
      color: #486ae8;
    }
  }
  .nearby-content {
    background: #ffffff;
    box-shadow: 2px 4px 7px rgba(196, 196, 196, 0.11);
    border-radius: 16px;
    margin-top: 40px;
    h2 {
      font-weight: bold;
      font-size: 20px;
      color: #7d7d7d;
      padding-top: 50px;
      padding-left: 70px;
    }
    .distance {
      padding-left: 70px;
      padding-right: 70px;
      padding-bottom: 50px;
      margin-top: 35px;
      .distance-title {
        display: flex;
        .distance-common {
          text-align: center;
          color: #ffffff;
          font-weight: bold;
          font-size: 18px;
          width: 50%;
          background: #bfcdff;
          padding: 12px 0;
          cursor: pointer;
          &:hover {
            transition: all 0.5s;
            background: #486ae8;
          }
        }
        .distance-100 {
          border-radius: 16px 0px 0px 0px;
          &.active {
            background: #486ae8;
          }
        }
        .distance-300 {
          border-radius: 0px 16px 0px 0px;
          &.active {
            background: #486ae8;
          }
        }
      }
      .distance-content {
        height: calc(100vh - 400px);
        overflow-y: scroll;
        .distance-txt {
          padding: 25px 0;
          padding-left: 70px;
          background: #f6f7ff;
          &:nth-child(2n) {
            background: #eceeff;
          }
          h3 {
            display: flex;
            font-weight: bold;
            font-size: 18px;
            color: #486ae8;
          }
          p {
            font-weight: 500;
            font-size: 18px;
            color: #7d7d7d;
            margin-top: 10px;
          }
        }
      }
    }
  }
}
</style>
