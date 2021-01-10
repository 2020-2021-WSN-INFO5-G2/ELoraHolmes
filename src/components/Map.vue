<template>
  <div id="map-wrapper">
    <v-alert type="error" v-if="showErr">{{ errContent }}</v-alert>
    <!--
    <div style="height: 300px overflow: auto;">
      <v-btn @click="showMap = !showMap">Toggle map</v-btn>
      {{center}}
    </div>
    -->
    <l-map v-if="showMap" :zoom="zoom" :center="center" :options="mapOptions">
      <l-tile-layer :url="url" :attribution="attribution" />
      <l-marker v-for="(values, key) in features" :key="key" :lat-lng="values">
        <l-popup>
          <div>
            {{ key }}
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
  props: ["frames", "gateways", "showError", "errorContent"],
  watch: {
    frames: {
      handler: function(newVal) {
        console.log(newVal);
        this.getFeatures();
      },
      deep: true
    },
    showError: {
      handler: function(newVal) {
        this.showErr = newVal;
      }
    },
    errorContent: {
      handler: function(newVal) {
        this.errContent = newVal;
      }
    }
  },
  data() {
    return {
      zoom: 15,
      features: {},
      center: latLng(45.187336, 5.7357),
      url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
      attribution:
        '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      showParagraph: false,
      mapOptions: {
        //zoomSnap: 0.5,
      },
      showMap: true,
      showErr: false,
      errContent: ""
    };
  },
  methods: {
    getFeatures() {
      for (var cle of Object.keys(this.frames)) {
        if (this.frames[cle].length > 0) {
          console.log("Feature to display : " + cle);
          axios({
            method: "post",
            url: "http://localhost:3000/loracloud/singleframe",
            headers: {
              Accept: "application/json",
              "Ocp-Apim-Subscription-Key": process.env.VUE_APP_LORACLOUD_KEY
            },
            data: {
              device: cle,
              gateways: this.gateways[cle],
              frames: this.frames[cle],
              params: {
                locAlgType: "RSSI_ALG", // "TDOA_ALG" or "RSSI_ALG"
                doRssiTdoaCombine: true
              }
            }
          })
            .then(res => {
              this.features[res.data.name] = {};
              this.features[res.data.name] = latLng(
                res.data.coordinates.latitude,
                res.data.coordinates.longitude
              );
              this.center = this.features[res.data.name];
              setTimeout(() => {
                this.zoom = 18;
              }, 500);
            })
            .catch(err => {
              this.showErr = true;
              this.errContent += err + " (" + err.response.data.error + "). ";
            });
        } else {
          if (this.features[cle]) {
            delete this.features[cle];
            this.center = latLng(45.187336, 5.7357);
            setTimeout(() => {
              this.zoom = 15;
            }, 500);
          }
        }
      }
    }
  }
};
</script>
<style scoped>
#map-wrapper {
  height: 100%;
  min-height: 500px;
  margin: 0;
}
</style>
