<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <div class="d-flex align-center">
        <v-img
          alt="Vuetify Logo"
          class="shrink mr-2"
          contain
          src="https://cdn.vuetifyjs.com/images/logos/vuetify-logo-dark.png"
          transition="scale-transition"
          width="40"
        />

        <v-img
          alt="Vuetify Name"
          class="shrink mt-1 hidden-sm-and-down"
          contain
          min-width="100"
          src="https://cdn.vuetifyjs.com/images/logos/vuetify-name-dark.png"
          width="100"
        />
      </div>

      <v-spacer></v-spacer>

      <v-btn
        href="https://github.com/vuetifyjs/vuetify/releases/latest"
        target="_blank"
        text
      >
        <span class="mr-2">Latest Release</span>
        <v-icon>mdi-open-in-new</v-icon>
      </v-btn>
    </v-app-bar>

    <v-main>
      <v-row>
        <v-col cols="12" md="2">
          <v-checkbox
            v-for="(values, key) in devices"
            :key="key"
            v-model="devices[key]"
            :label="key"
            :value="values"
          ></v-checkbox>
          <v-btn depressed v-on:click="search"> Localiser </v-btn>
        </v-col>

        <v-col cols="12" md="10">
          <Map :frames="frames" :gateways="gateways" />
        </v-col>
      </v-row>
    </v-main>
  </v-app>
</template>

<script>
import Map from "./components/Map";
import axios from "axios";

export default {
  name: "App",

  components: {
    Map,
  },

  data() {
    return {
      devices: {
        "explorer-sherlock-board": false,
        "elora-sodaq-romain": false,
        "explorer-sherlock-nucleo": false,
        devCartan: false,
      },
      frames: {
        "explorer-sherlock-board": [],
        "elora-sodaq-romain": [],
        "explorer-sherlock-nucleo": [],
        devCartan: [],
      },
      gateways: {
        "explorer-sherlock-board": [],
        "elora-sodaq-romain": [],
        "explorer-sherlock-nucleo": [],
        devCartan: [],
      },
    };
  },
  methods: {
    search() {
      for (var [cle, valeur] of Object.entries(this.devices)) {
        let data = [];
        if (valeur) {
          axios
            .get("http://localhost:3000/devices/" + cle)
            .then((response) => {
              data = response.data;
              data.forEach((element) => {
                //DEBUG
                if (element.metadata !== undefined)
                  console.log(element.metadata);

                //Storing Gateways and Frames infos
                if (this.gateways[element.dev_id] === undefined)
                  this.gateways[element.dev_id] = [];
                if (this.frames[element.dev_id] === undefined)
                  this.frames[element.dev_id] = [];

                element.metadata.gateways.forEach((g) => {
                  if (g.latitude && g.altitude && g.longitude) {
                    this.gateways[element.dev_id].push({
                      gatewayId: g.gtw_id,
                      latitude: g.latitude,
                      longitude: g.longitude,
                      altitude: g.altitude,
                    });
                  }
                  if (g.rssi && g.snr) {
                    this.frames[element.dev_id].push([
                      g.gtw_id,
                      g.antenna || null,
                      g.timestamp || null,
                      g.rssi,
                      g.snr,
                    ]);
                  }
                });
              });
            })
            .catch((err) => {
              console.log(err);
            });
        }
      }
    },
  },
};
</script>
