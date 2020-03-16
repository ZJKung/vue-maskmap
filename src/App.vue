<template>
  <div id="app">
    <div class="row no-gutters">
      <div class="col-sm-3">
        <div class="toolbox">
          <div class="sticky-top bg-white shadow-sm p-2">
            <div class="form-group d-flex">
              <label for="cityName" class="mr-2 col-form-label text-right">縣市</label>
              <div class="flex-fill">
                <select class="form-control" v-model="selectedCity">
                  <option value>-----Select City</option>
                  <option
                    :value="city.CityName"
                    v-for="city in cityData"
                    :key="city.CityName"
                  >{{city.CityName}}</option>
                </select>
              </div>
            </div>
            <div class="form-group d-flex">
              <label for="area" class="mr-2 col-form-label text-right">地區</label>
              <div class="flex-fill">
                <select class="form-control" v-model="selectedRegion">
                  <option value>-- Select One --</option>
                  <option
                    :value="r.AreaName"
                    v-for="r in regionData"
                    :key="r.AreaName"
                  >{{r.AreaName}}</option>
                </select>
              </div>
            </div>
            <p class="mb-0 small text-muted text-right">請先選擇區域查看（綠色表示還有口罩）</p>
          </div>
          <ul class="list-group">
            <div v-for="s in stores" :key="s.properties.id">
              <a class="list-group-item text-left">
                <h3>{{s.properties.name}}</h3>
                <p class="mb-0">成人口罩：{{s.properties.mask_adult}} | 兒童口罩：{{s.properties.mask_child}}</p>
                <p class="mb-0">
                  地址：
                  <a
                    :href="'https://www.google.com.tw/maps/place/'+s.properties.address"
                    target="_blank"
                    title="Google Map"
                  >{{s.properties.address}}</a>
                </p>
              </a>
            </div>
          </ul>
        </div>
      </div>
      <div class="col-sm-9">
        <div id="map"></div>
      </div>
    </div>
  </div>
</template>

<script>
import L from "leaflet";
console.log(L);
//let mymap;
export default {
  name: "App",
  data: () => ({
    pharmaciesData: [],
    stores: [],
    mymap: null,
    cityData: null,
    regionData: [],
    selectedCity: "",
    selectedRegion: ""
  }),
  components: {},
  watch: {
    selectedCity: function(value) {
      this.removeLabel();
      if (this.selectedCity) {
        this.regionData = this.cityData
          .filter(x => x.CityName == value)
          .flatMap(x => x.AreaList);
      }
      this.updateMap();
    },
    selectedRegion: function() {
      this.removeLabel();
      this.updateMap();
    }
  },
  methods: {
    updateMap() {
      if (this.selectedCity) {
        this.stores = this.pharmaciesData.filter(
          x => x.properties.county == this.selectedCity
        );
        if (this.selectedRegion) {
          this.stores = this.stores.filter(
            x => x.properties.town == this.selectedRegion
          );
        }
        this.stores.forEach(pharmacy => {
          const { properties, geometry } = pharmacy;

          //icon.iconUrl=properties.mask_adult>50?'./assets/icon/marker-icon-green.png':'./assets/icon/marker-icon-orange.png';
          //console.log(icon);

          L.marker([geometry.coordinates[1], geometry.coordinates[0]]).addTo(
            this.mymap
          ).bindPopup(`<strong>${properties.name}</strong> <br>
    口罩剩餘：<strong>成人 - ${
      properties.mask_adult ? `${properties.mask_adult} 個` : "未取得資料"
    }/ 兒童 - ${
            properties.mask_child ? `${properties.mask_child} 個` : "未取得資料"
          }</strong><br>
    地址: <a href="https://www.google.com.tw/maps/place/${
      properties.address
    }" target="_blank">${properties.address}</a><br>
    電話: ${properties.phone}<br>
    <small>最後更新時間: ${properties.updated}</small>`);
        });
      }
    },
    removeLabel() {
      this.mymap.eachLayer(layer => {
        if (layer instanceof L.Marker) {
          this.mymap.removeLayer(layer);
        }
      });
    }
  },
  created() {
    const pharmacyAPI =
      "https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json";
    const cityAPI =
      "https://raw.githubusercontent.com/donma/TaiwanAddressCityAreaRoadChineseEnglishJSON/master/CityCountyData.json";
    Promise.all([this.axios.get(pharmacyAPI), this.axios.get(cityAPI)]).then(
      ([pharmacyRes, cityRes]) => {
        this.pharmaciesData = pharmacyRes.data.features;
        this.cityData = cityRes.data;
      }
    );
  },
  mounted() {
    var self = this;
    self.mymap = L.map("map", {
          center: [24.156547, 120.712760],
          zoom: 12,
        });
    navigator.geolocation.getCurrentPosition(
      pos => {
        self.mymap.setView([pos.coords.latitude, pos.coords.longitude],15);
        L.marker([pos.coords.latitude, pos.coords.longitude]).addTo(self.mymap)
        .bindPopup("<h4>目前位置</h4>")
        .openPopup();
      },
      err => {
        console.error(err);
      },
      {
        enableHighAccuracy: true,
        timeout: 5000,
        maximumAge: 0
      }
    );
    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution:
        'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>',
      maxZoom: 18
    }).addTo(self.mymap);
    
  }
};
</script>

<style lang="scss">
@import "bootstrap/scss/bootstrap";
#map {
  height: 100vh;
}
.highlight {
  background: #e9ffe3;
}
.toolbox {
  height: 100vh;
  overflow-y: auto;
  a {
    cursor: pointer;
  }
}
</style>