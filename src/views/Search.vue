<template>
  <Loading :active="isLoading" />
  <Navbar />
  <div class="search">
    <Title />
    <div class="search-content">
      <div class="search-left">
        <div class="search-meter">
          <p>Where are you going?</p>
          <select v-model="currentCategory" @change="getBusData">
            <option value="">請選擇縣市</option>
            <option :value="item.City" v-for="item in city" :key="item">
              {{ item.CityName }}
            </option>
          </select>
          <select v-model="currentRoute">
            <option value="">請選擇路線</option>
            <option :value="item.RouteName.Zh_tw" v-for="item in busRoute" :key="item">
              {{ item.RouteName.Zh_tw }}
            </option>
          </select>
          <button type="button" @click="setBusData(currentRoute)">搜尋</button>
        </div>
        <div class="search-time">
          <div class="search-time-title">
            <div class="search-time-common search-time-go active">往 公教住宅</div>
            <div class="search-time-common search-time-back">往 中和</div>
          </div>
          <div class="search-time-content">
            <div class="search-time-txt">
              <img src="../assets/img/icon-down.png" alt="down" />
              <h3>中和保養廠</h3>
              <p class="time">未發車</p>
              <div class="car"><img src="../assets/img/icon-car.png" alt="car" />FAA-073</div>
            </div>
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

let map = {};
let lineLayer = {};
const redIcon = new L.Icon({
  iconUrl: 'https://cdn.rawgit.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-red.png',
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
      busEstimate: [],
      currentCategory: '',
      currentRoute: '',
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
            L.marker([this.userLocation.latitude, this.userLocation.longitude])
              .addTo(map)
              .bindPopup('您的位置')
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
    getBusData() {
      this.getData(
        `https://ptx.transportdata.tw/MOTC/v2/Bus/Route/City/${this.currentCategory}?$format=JSON`,
        'busRoute',
      );
      this.getData(
        `https://ptx.transportdata.tw/MOTC/v2/Bus/StopOfRoute/City/${this.currentCategory}?$format=JSON`,
        'busSite',
      );
      this.getData(
        `https://ptx.transportdata.tw/MOTC/v2/Bus/Shape/City/${this.currentCategory}?$format=JSON`,
        'busLine',
      );
      this.getData(
        `https://ptx.transportdata.tw/MOTC/v2/Bus/RealTimeByFrequency/City/${this.currentCategory}?$format=JSON`,
        'busRealTime',
      );
      this.getData(
        `https://ptx.transportdata.tw/MOTC/v2/Bus/EstimatedTimeOfArrival/City/${this.currentCategory}?$format=JSON`,
        'busEstimate',
      );
    },
    setBusData(name) {
      const filterBusSite = this.busSite.filter((item) => item.RouteName.Zh_tw === name);
      const filterBusLine = this.busLine.filter((item) => item.RouteName.Zh_tw === name);
      const filterBusRealTime = this.busRealTime.filter((item) => item.RouteName.Zh_tw === name);
      // const filterBusEstimate = this.busEstimate.filter((item) => item.RouteName.Zh_tw === name);
      this.removeMarker();
      this.renderSiteMarker(filterBusSite);
      this.drawLine(filterBusLine[0].Geometry);
      this.renderBusMarker(filterBusRealTime);
    },
    renderSiteMarker(data) {
      data[0].Stops.forEach((item) => {
        L.marker([item.StopPosition.PositionLat, item.StopPosition.PositionLon]).addTo(map);
      });
      map.setView([
        data[0].Stops[0].StopPosition.PositionLat,
        data[0].Stops[0].StopPosition.PositionLon,
      ]);
    },
    renderBusMarker(data) {
      data.forEach((item) => {
        L.marker([item.BusPosition.PositionLat, item.BusPosition.PositionLon], {
          icon: redIcon,
        }).addTo(map);
      });
      // map.setView([
      //   data[0].Stops[0].StopPosition.PositionLat,
      //   data[0].Stops[0].StopPosition.PositionLon,
      // ]);
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
        color: '#07B041',
        weight: 5,
        opacity: 0.5,
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

    const searchTimeGo = document.querySelector('.search-time-go');
    const searchTimeBack = document.querySelector('.search-time-back');
    searchTimeGo.addEventListener('click', () => {
      searchTimeGo.classList.add('active');
      searchTimeBack.classList.remove('active');
    });
    searchTimeBack.addEventListener('click', () => {
      searchTimeGo.classList.remove('active');
      searchTimeBack.classList.add('active');
    });
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
        margin-left: 30px;
      }
      select {
        font-weight: bold;
        font-size: 16px;
        color: #363636;
        margin-left: 30px;
        margin-right: 30px;
        margin-top: 30px;
        &:focus{
          outline: none;
        }
      }
      button {
        /* justify-self: center; */
        align-self: center;
        width: 275px;
        color: #ffffff;
        background: #486ae8;
        border-radius: 32px;
        padding: 9px 120px;
        margin-top: 35px;
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
          justify-content: center;
          align-items: center;
          padding: 10px 0;
          &:first-child {
            margin-top: 0;
          }
          &:nth-child(2n) {
            background: #fefcfc;
          }
          h3 {
            font-weight: normal;
            font-size: 16px;
            color: #7d7d7d;
            padding-left: 45px;
          }
          .time {
            font-weight: bold;
            font-size: 16px;
            color: #7d7d7d;
            padding-left: 45px;
          }
          .car {
            display: flex;
            justify-content: center;
            align-items: center;
            padding-left: 45px;
            font-weight: normal;
            font-size: 14px;
            color: #486ae8;
            img {
              margin-right: 15px;
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
