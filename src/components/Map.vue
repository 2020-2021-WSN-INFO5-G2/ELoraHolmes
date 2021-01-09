<template>
  <div style="height: 500px; width: 100%">
    <div style="height: 200px overflow: auto;">
      <v-btn @click="showMap = !showMap">Toggle map</v-btn>
    </div>
    <l-map
      v-if="showMap"
      :zoom="zoom"
      :center="center"
      :options="mapOptions"
      style="height: 80%"
      @update:center="centerUpdate"
      @update:zoom="zoomUpdate"
    >
      <l-tile-layer :url="url" :attribution="attribution" />
      <l-marker v-for="(values, key) in features"
        :key="key" :lat-lng="values">
        <l-popup>
          <div>
            {{key}}
          </div>
        </l-popup>
      </l-marker>
    </l-map>
  </div>
</template>

<script>
import { latLng } from "leaflet";
import axios from "axios";

export default {
  name: "Map",
  components: {},
  props: ["frames", "gateways"],
  watch: {
    frames: {
      handler: function (newVal) {
        console.log(newVal);
        this.getFeatures();
      },
      deep: true,
    },
  },
  data() {
    return {
      zoom: 13,
      features: {},
      center: latLng(47.41322, -1.219482),
      url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
      attribution:
        '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      currentZoom: 11.5,
      currentCenter: latLng(47.41322, -1.219482),
      showParagraph: false,
      mapOptions: {
        zoomSnap: 0.5,
      },
      showMap: true,
    };
  },
  methods: {
    zoomUpdate(zoom) {
      this.currentZoom = zoom;
    },
    centerUpdate(center) {
      this.center = center;
    },
    getFeatures() {
      for (var cle of Object.keys(this.frames)) {
        if (this.frames[cle].length != 0) {
          axios({
            method: "post",
            url: "http://localhost:3000/loracloud/singleframe",
            headers: {
              Accept: "application/json",
              "Ocp-Apim-Subscription-Key": process.env.VUE_ENV_LORACLOUD_KEY,
            },
            data: {
              device: cle,
              gateways: this.gateways[cle],
              frames: this.frames[cle],
              params: {
                locAlgType: "RSSI_ALG", // "TDOA_ALG" or "RSSI_ALG"
                doRssiTdoaCombine: true,
              },
            },
          }).then((res) => {
            if (this.features[res.data.name] === undefined)
                  this.features[res.data.name] = {};
            this.features[res.data.name] = latLng(res.data.coordinates.latitude,res.data.coordinates.longitude);
            this.centerUpdate(this.features[res.data.name]);
          });
        }
      }
    },
  },
};
</script>
