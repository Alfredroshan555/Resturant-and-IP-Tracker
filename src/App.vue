<template>
  <v-app>
    <div>
      <h1>hello</h1>
      <v-app-bar color="deep-purple accent-4" dense dark>
        <v-app-bar-nav-icon></v-app-bar-nav-icon>

        <v-toolbar-title>Resturant App</v-toolbar-title>

        <v-spacer></v-spacer>
        <div class="ipInput">
          <v-text-field
            label="Enter IP address"
            v-model="input"
            @keypress.enter="getIp"
          ></v-text-field>
        </div>

        <v-btn @click="getLocation">
          <v-icon>mdi-magnify</v-icon>
        </v-btn>

        <v-menu left bottom>
          <template v-slot:activator="{ on, attrs }">
            <v-btn icon v-bind="attrs" v-on="on">
              <v-icon>mdi-dots-vertical</v-icon>
            </v-btn>
          </template>
        </v-menu>
      </v-app-bar>
    </div>
    <v-main>
      <!-- <div class="data" v-for="data in datas" :key="data.id">
        <h3>{{ data.id }}</h3>
        <h4>{{ data.city }}</h4>
        <h4>lat--{{ data.lat }}</h4>
      </div> -->

      <div id="map"></div>
      <div class="card-scroll">
        <div class="card-details" v-for="data in datas" :key="data.id">
          <v-card :loading="loading" class="card" max-width="374">
            <template slot="progress"> </template>
            <!-- <v-title>Top Resturants in US</v-title> -->
            <v-img
              height="250"
              src="https://cdn.vuetifyjs.com/images/cards/cooking.png"
            ></v-img>

            <v-card-title>{{ data.name }}</v-card-title>

            <v-card-text>
              <v-row align="center" class="mx-0">
                <v-rating
                  :value="4.5"
                  color="purple"
                  dense
                  half-increments
                  readonly
                  size="14"
                ></v-rating>
              </v-row>

              <div class="my-4 text-subtitle-1">
                {{ data.city }} || {{ data.state }}
              </div>

              <h4>{{ data.country }}</h4>

              <div>Address : {{ data.street }}</div>
              <div>Latitide -- {{ data.latitude }}</div>
              <div>Longitude -- {{ data.longitude }}</div>
            </v-card-text>

            <v-card-title> Contact No : {{ data.phone }}</v-card-title>
          </v-card>
        </div>
      </div>
    </v-main>
  </v-app>
</template>

<script>
import leaflet from "leaflet";
import axios from "axios";

export default {
  name: "App",

  components: {},
  data() {
    return {
      map: "",
      datas: [],
      input: "",
      marker: [],
      latitude: [],
      longitude: [],
      city: "",
      country: "",
      street: "",
      phone: "",
      latIp: "",
      longIp: "",
      name: "",
      state: "",
      popup: "",
    };
  },
  //setting up map
  mounted() {
    this.map = leaflet.map("map").setView([10.5276, 76.2144], 9);
    leaflet.Control.geocoder().addTo(this.map);
    leaflet
      .tileLayer(
        "https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=pk.eyJ1IjoiYWxmcmVkNTU1IiwiYSI6ImNreDcxMXh4NDMxNG4yb3A4Z2lheDR2Z2cifQ.T3oUPe2RMArV_v7eduyl0A",
        {
          attribution:
            'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
          maxZoom: 18,
          id: "mapbox/streets-v11",
          tileSize: 512,
          zoomOffset: -1,
          accessToken:
            "pk.eyJ1IjoiYWxmcmVkNTU1IiwiYSI6ImNreDcxMXh4NDMxNG4yb3A4Z2lheDR2Z2cifQ.T3oUPe2RMArV_v7eduyl0A",
        }
      )
      .addTo(this.map);
    this.popup.openPopup();
  },
  methods: {
    //getting resturant data from api
    async getLocation() {
      await axios.get(`https://api.openbrewerydb.org/breweries`).then((res) => {
        this.datas = res.data;
        this.city = res.data.city;
        this.latitude = res.data.latitude;
        this.longitude = res.data.longitude;
        this.country = res.data.country;
        this.street = res.data.street;
        this.phone = res.data.phone;
        this.name = res.data.name;
        this.state = res.data.state;
        console.log(this.datas);
        console.log(this.city);
        console.log(this.latitude);
        console.log(this.longitude);
      });
      leaflet
        .marker([this.latitude, this.longitude])
        .addTo(this.map)
        .bindPopup("A pretty CSS3 popup.<br> Easily customizable.")
        .openPopup();
      this.map.setView([this.latitude, this.longitude], 13);
    },
    //getting ip info from api
    async getIp() {
      await axios
        .get(
          `https://geo.ipify.org/api/v1?apiKey=at_FBTi0rGPhQlKp2cRdOJV0t3Mibq6F&ipAddress=${this.input}`
        )
        .then((res) => {
          this.datas = res.data;
          this.city = res.data.location.city;
          this.latIp = res.data.location.lat;
          this.longIp = res.data.location.lng;
          console.log(this.datas);
          console.log(this.latIp);
          console.log(this.longIp);
        });
      leaflet
        .marker([this.latIp, this.longIp])
        .addTo(this.map)
        .bindPopup("A pretty CSS3 popup.<br> Easily customizable.")
        .openPopup();
      this.map.setView([this.latIp, this.longIp], 13);
    },
  },
};
</script>

<style scoped>
body {
  max-width: 100%;
  max-height: 100%;
  overflow: hidden;
}
#map {
  height: 100%;
  width: 75%;

  float: right;
}
.data {
  background-color: aqua;
  width: 36%;
}
.card-details {
  width: 100%;
}
.ipInput {
  padding: 30px;
  padding-bottom: 20px;
  padding-right: 150px;
}
.card-scroll {
  overflow: scroll;
  height: 100vh;
}
</style>
