<template>
  <Loading :active="isLoading" style="z-index: 9999" />
  <Navbar />
  <div class="search">
    <Title />
    <div class="search-content">
      <div class="search-left">
        <div class="search-meter">
          <p>Where are you going?</p>
          <select v-model="currentCategory" @change="getBusRoute">
            <option value="">請選擇縣市</option>
            <option :value="item.City" v-for="item in city" :key="item">
              {{ item.CityName }}
            </option>
          </select>
          <select v-model="currentRoute" @change="getBusData">
            <option value="">請選擇路線</option>
            <option :value="item.RouteName.Zh_tw" v-for="item in busRoute" :key="item">
              {{ item.RouteName.Zh_tw }}
            </option>
          </select>
          <div class="search-meter-btn">
            <button type="button" @click="setBusData">搜尋</button
            ><button type="button" @click="resetBusData">重新整理</button>
          </div>
        </div>
        <div class="search-time">
          <div class="search-time-title">
            <div
              class="search-time-common search-time-go"
              :class="{ active: status === 'go' }"
              @click="setBusGoData"
            >
              往 {{ busGoName }}
            </div>
            <div
              class="search-time-common search-time-back"
              :class="{ active: status === 'back' }"
              @click="setBusBackData"
            >
              往 {{ busBackName }}
            </div>
          </div>
          <div class="search-time-content">
            <template v-if="status === 'go'">
              <div class="search-time-txt" v-for="item in busGoEstimate" :key="item">
                <img src="../assets/img/icon-down.png" alt="down" />
                <h3>{{ item.StopName.Zh_tw }}</h3>
                <p class="time" v-if="item.StopStatus === 0">
                  {{ Math.round(item.EstimateTime / 60) }} 分
                </p>
                <p class="time" v-if="item.StopStatus === 1">尚未發車</p>
                <div class="car"  v-if="item.StopStatus === 0">
                  <img src="../assets/img/icon-car.png" alt="car" />{{ item.PlateNumb }}
                </div>
                <div class="car" v-if="item.StopStatus === 1">
                </div>
              </div>
            </template>
            <template v-if="status === 'back'">
              <div class="search-time-txt" v-for="item in busBackEstimate" :key="item">
                <img src="../assets/img/icon-down.png" alt="down" />
                <h3>{{ item.StopName.Zh_tw }}</h3>
                <p class="time" v-if="item.StopStatus === 0">
                  {{ Math.round(item.EstimateTime / 60) }} 分
                </p>
                <p class="time" v-if="item.StopStatus === 1">尚未發車</p>
                <div class="car" v-if="item.StopStatus === 0">
                  <img src="../assets/img/icon-car.png" alt="car" />{{ item.PlateNumb }}
                </div>
                <div class="car" v-if="item.StopStatus === 1">
                </div>
              </div>
            </template>
          </div>
        </div>
      </div>
      <div class="search-right">
        <div id="mapid" class="map"></div>
      </div>
    </div>
    <Footer />
  </div>
</template>

<script>
import L from 'leaflet';
import Wkt from 'wicket';
import 'leaflet.markercluster/dist/leaflet.markercluster';

import getAuthorizationHeader from '../tools/AuthorizationHeader';
import Navbar from '../components/Navbar.vue';
import Title from '../components/Title.vue';
import Footer from '../components/Footer.vue';
import userIconImg from '../assets/img/icon-user.png';

let map = {};
let lineLayer = {};
const userIcon = new L.Icon({
  iconUrl: userIconImg,
  iconSize: [46, 46],
  iconAnchor: [20, 0],
});
const redIcon = new L.Icon({
  iconUrl: 'https://cdn.rawgit.com/pointhi/leaflet-color-markers/master/img/marker-icon-red.png',
  shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
  iconSize: [25, 41],
  iconAnchor: [12, 41],
  popupAnchor: [1, -34],
  shadowSize: [41, 41],
});

export default {
  name: 'Search',
  components: {
    Navbar,
    Title,
    Footer,
  },
  data() {
    return {
      userLocation: {
        longitude: 0,
        latitude: 0,
      },
      city: [],
      busRoute: [],
      busSite: [],
      busLine: [],
      busRealTime: [],
      busGoRealTime: [],
      busBackRealTime: [],
      busEstimate: [],
      busGoEstimate: [],
      busBackEstimate: [],
      currentCategory: '',
      currentRoute: '',
      busGoName: '',
      busBackName: '',
      status: 'go',
      isLoading: false,
    };
  },
  methods: {
    getUserLocation() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          (position) => {
            this.userLocation.longitude = position.coords.longitude;
            this.userLocation.latitude = position.coords.latitude;
            map.setView([this.userLocation.latitude, this.userLocation.longitude]);
            map.panTo([this.userLocation.latitude, this.userLocation.longitude]);
            L.marker([this.userLocation.latitude, this.userLocation.longitude], {
              icon: userIcon,
            })
              .addTo(map)
              .bindPopup('您的位置', { closeButton: false })
              .openPopup();
          },
          (e) => {
            const msg = e.message;
            console.error(msg);
          },
        );
      }
    },
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
    getBusRoute() {
      this.getData(
        `https://ptx.transportdata.tw/MOTC/v2/Bus/Route/City/${this.currentCategory}/${this.currentRoute}?$format=JSON`,
        'busRoute',
      );
    },
    getBusData() {
      this.getData(
        `https://ptx.transportdata.tw/MOTC/v2/Bus/StopOfRoute/City/${this.currentCategory}/${this.currentRoute}?$format=JSON`,
        'busSite',
      );
      this.getData(
        `https://ptx.transportdata.tw/MOTC/v2/Bus/Shape/City/${this.currentCategory}/${this.currentRoute}?$format=JSON`,
        'busLine',
      );
      this.getTimelyBusData();
    },
    getTimelyBusData() {
      this.getData(
        `https://ptx.transportdata.tw/MOTC/v2/Bus/RealTimeByFrequency/City/${this.currentCategory}/${this.currentRoute}?$format=JSON`,
        'busRealTime',
      );
      this.getData(
        `https://ptx.transportdata.tw/MOTC/v2/Bus/EstimatedTimeOfArrival/City/${this.currentCategory}/${this.currentRoute}?$format=JSON`,
        'busEstimate',
      );
    },
    setBusData() {
      const filterBusEstimate0 = this.busEstimate.filter((item) => item.Direction === 0);
      this.busGoEstimate = filterBusEstimate0.sort((a, b) => a.StopSequence - b.StopSequence);
      const filterBusEstimate1 = this.busEstimate.filter((item) => item.Direction === 1);
      this.busBackEstimate = filterBusEstimate1.sort((a, b) => a.StopSequence - b.StopSequence);
      this.busGoRealTime = this.busRealTime.filter((item) => item.Direction === 0);
      this.busBackRealTime = this.busRealTime.filter((item) => item.Direction === 1);
      this.busGoName = this.busBackEstimate[0].StopName.Zh_tw;
      this.busBackName = this.busGoEstimate[0].StopName.Zh_tw;
      if (this.status === 'go') {
        this.setBusGoData();
      } else if (this.status === 'back') {
        this.setBusBackData();
      }
    },
    resetBusData() {
      if (this.currentCategory === '' || this.currentRoute === '') return;
      this.getTimelyBusData();
      setTimeout(() => {
        this.setBusData();
      }, 1000);
    },
    setBusGoData() {
      this.status = 'go';
      this.removeMarker();
      this.renderSiteMarker(this.busSite[0]);
      this.drawLine(this.busLine[0].Geometry);
      this.renderBusMarker(this.busGoRealTime);
    },
    setBusBackData() {
      this.status = 'back';
      this.removeMarker();
      this.renderSiteMarker(this.busSite[1]);
      this.drawLine(this.busLine[1].Geometry);
      this.renderBusMarker(this.busBackRealTime);
    },
    renderSiteMarker(data) {
      data.Stops.forEach((item, index) => {
        L.marker([item.StopPosition.PositionLat, item.StopPosition.PositionLon])
          .addTo(map)
          .bindTooltip(`${index + 1}. ${data.Stops[index].StopName.Zh_tw}`, {
            permanent: true,
            direction: 'top',
            offset: [-15, -10],
          })
          .openTooltip();
      });
    },
    renderBusMarker(data) {
      data.forEach((item) => {
        L.marker([item.BusPosition.PositionLat, item.BusPosition.PositionLon], { icon: redIcon })
          .addTo(map)
          .bindTooltip(`${item.PlateNumb}`, {
            permanent: true,
            direction: 'bottom',
            offset: [0, 0],
          })
          .openTooltip();
      });
    },
    removeMarker() {
      map.eachLayer((layer) => {
        if (layer instanceof L.Marker) {
          map.removeLayer(layer);
        }
      });
    },
    drawLine(geo) {
      map.removeLayer(lineLayer);
      const wicket = new Wkt.Wkt();
      const geoTojson = wicket.read(geo).toJson();
      const styleLine = {
        color: '#3356D6',
        weight: 4,
        opacity: 1,
      };
      lineLayer = L.geoJSON(geoTojson, {
        style: styleLine,
      }).addTo(map);
      lineLayer.addData(geoTojson);
      map.fitBounds(lineLayer.getBounds());
    },
  },
  mounted() {
    this.getUserLocation();
    this.getData('https://gist.motc.gov.tw/gist_api/V3/Map/Basic/City?$format=JSON', 'city');

    map = L.map('mapid').setView([0, 0], 14);
    L.tileLayer(
      'https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token={accessToken}',
      {
        attribution:
          'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery &copy; <a href="https://www.mapbox.com/">Mapbox</a>',
        maxZoom: 18,
        id: 'mapbox/streets-v11',
        tileSize: 512,
        zoomOffset: -1,
        accessToken: process.env.VUE_APP_MAPKEY,
      },
    ).addTo(map);
  },
};
</script>

<style lang="scss" scoped>
.map {
  width: 100%;
  height: calc(100vh - 190px);
}
.search {
  width: calc(100% - 142px);
  padding: 55px 80px 0 80px;
  .search-content {
    width: 100%;
    margin-top: 40px;
    display: flex;
  }
  .search-left {
    width: 30%;
    display: flex;
    flex-direction: column;
    margin-right: 30px;
    .search-meter {
      background: #ffffff;
      box-shadow: 2px 4px 7px rgba(196, 196, 196, 0.11);
      border-radius: 16px;
      display: flex;
      flex-direction: column;
      padding-bottom: 30px;
      p {
        font-family: 'DM Sans', sans-serif;
        font-weight: bold;
        font-size: 24px;
        color: #7d7d7d;
        padding-top: 40px;
        margin-left: 20px;
      }
      select {
        font-weight: bold;
        font-size: 15px;
        color: #363636;
        margin-left: 20px;
        margin-right: 20px;
        margin-top: 30px;
        &:focus {
          outline: none;
        }
      }
      .search-meter-btn {
        display: flex;
        justify-content: space-evenly;
        button {
          align-self: center;
          color: #ffffff;
          background: #486ae8;
          border-radius: 32px;
          padding: 9px 50px;
          margin-top: 35px;
          cursor: pointer;
          transition: all 0.5s;
          &:hover {
            background-color: #2a4ac3;
          }
        }
      }
    }
    .search-time {
      background: #ffffff;
      box-shadow: 2px 4px 7px rgba(196, 196, 196, 0.11);
      border-radius: 16px;
      margin-top: 35px;
      .search-time-title {
        display: flex;
        .search-time-common {
          text-align: center;
          color: #ffffff;
          font-weight: bold;
          font-size: 15px;
          width: 50%;
          background: #bfcdff;
          padding: 12px 0;
          cursor: pointer;
          transition: all 0.5s;
          &:hover {
            background: #486ae8;
          }
        }
        .search-time-go {
          border-radius: 16px 0px 0px 0px;
          &.active {
            background: #486ae8;
          }
        }
        .search-time-back {
          border-radius: 0px 16px 0px 0px;
          &.active {
            background: #486ae8;
          }
        }
      }
      .search-time-content {
        height: calc(100vh - 548px);
        overflow-y: scroll;
        border: 1px solid #f3f3f3;
        > h3 {
          font-weight: bold;
          font-size: 18px;
          color: #486ae8;
          padding: 25px 0;
          padding-left: 70px;
        }
        .search-time-txt {
          background: #f2f2f2;
          display: flex;
          justify-content: space-evenly;
          align-items: center;
          padding: 10px 0px;
          &:first-child {
            margin-top: 0;
          }
          &:nth-child(2n) {
            background: #fefcfc;
          }
          h3 {
            width: 235px;
            text-align: center;
            font-weight: normal;
            font-size: 16px;
            color: #7d7d7d;
          }
          .time {
            width: 65px;
            text-align: center;
            font-weight: bold;
            font-size: 16px;
            color: #7d7d7d;
          }
          .car {
            width: 85px;
            display: flex;
            justify-content: center;
            align-items: center;
            font-weight: normal;
            font-size: 14px;
            color: #486ae8;
            img {
              margin-right: 10px;
            }
          }
        }
      }
    }
  }
  .search-right {
    width: 70%;
  }
}
</style>
