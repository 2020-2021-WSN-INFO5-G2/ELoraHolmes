<template>
  <v-app>
    <v-app-bar app color="deep-purple" dark>
      <v-app-bar-nav-icon @click="drawer = true"></v-app-bar-nav-icon>
      <div class="d-flex align-center">
        <v-img
          alt="Vuetify Logo"
          class="shrink mr-2"
          contain
          src="https://cdn.vuetifyjs.com/images/logos/vuetify-logo-dark.png"
          transition="scale-transition"
          width="40"
        />

        <v-toolbar-title class="hidden-sm-and-down"
          >ELora Holmes</v-toolbar-title
        >
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
    <v-navigation-drawer
      v-model="drawer"
      absolute
      temporary
      style="z-index: 9999"
    >
      <v-list nav dense>
        <v-list-item>
          <v-list-item-icon>
            <v-icon>mdi-devices</v-icon>
          </v-list-item-icon>
          <v-list-item-title>Devices</v-list-item-title>
        </v-list-item>
        <v-list-item-group
          v-model="group"
          active-class="deep-purple--text text--accent-4"
        >
          <v-list-item v-for="(values, key) in devices" :key="key">
            <v-checkbox
              v-model="devices[key]"
              :label="key"
              :value="values"
            ></v-checkbox>
          </v-list-item>
          <v-list-item
            ><v-btn depressed v-on:click="search">
              Localiser
            </v-btn></v-list-item
          >
          <v-list-item
            ><v-btn depressed v-on:click="stopsearching">
              Arrêter la recherche
            </v-btn></v-list-item
          >
        </v-list-item-group>
      </v-list>
    </v-navigation-drawer>

    <v-main>
      <Map
        :frames="frames"
        :gateways="gateways"
        :showError="showError"
        :errorContent="errorContent"
      />
    </v-main>
    <v-footer padless>
      <v-col class="text-center" cols="12">
        {{ new Date().getFullYear() }} —
        <strong>Myriam Lombard, Romain Pasdeloup</strong>
      </v-col>
    </v-footer>
  </v-app>
</template>

<script>
import Map from "./components/Map";
import axios from "axios";

export default {
  name: "App",

  components: {
    Map
  },

  data() {
    return {
      drawer: false,
      group: null,
      devices: {
        "explorer-sherlock-board": false,
        "elora-sodaq-romain": false,
        "explorer-sherlock-nucleo": false,
        "fake-device": false
      },
      frames: {
        "explorer-sherlock-board": [],
        "elora-sodaq-romain": [],
        "explorer-sherlock-nucleo": [],
        "fake-device": []
      },
      gateways: {
        "explorer-sherlock-board": [],
        "elora-sodaq-romain": [],
        "explorer-sherlock-nucleo": [],
        "fake-device": []
      },
      showError: false,
      errorContent: ""
    };
  },
  methods: {
    search() {
      this.showError = false;
      this.errorContent = "";
      for (var [cle, valeur] of Object.entries(this.devices)) {
        let data = [];
        if (valeur) {
          axios
            .get("http://localhost:3000/devices/" + cle)
            .then(response => {
              data = response.data.frames;
              axios({
                method: "post",
                url: "http://localhost:3000/devices/" + response.data.device,
                data: {
                  ledstatus: true
                }
              }).then(res => {
                console.log("LED Up: " + res.data.message);
              });
              data.forEach(element => {
                //Storing Gateways and Frames infos
                this.gateways[element.dev_id] = [];
                this.frames[element.dev_id] = [];

                element.metadata.gateways.forEach(g => {
                  if (g.latitude && g.altitude && g.longitude) {
                    this.gateways[element.dev_id].push({
                      gatewayId: g.gtw_id,
                      latitude: g.latitude,
                      longitude: g.longitude,
                      altitude: g.altitude
                    });
                  }
                  if (g.rssi && g.snr) {
                    this.frames[element.dev_id].push([
                      g.gtw_id,
                      g.antenna || null,
                      g.timestamp || null,
                      g.rssi,
                      g.snr
                    ]);
                  }
                });
              });
            })
            .catch(err => {
              this.showError = true;
              this.errorContent += err + " (" + err.response.data.error + "). ";
            });
        }
      }
      this.drawer = false;
    },
    stopsearching() {
      for (var [cle, valeur] of Object.entries(this.devices)) {
        if (valeur) {
          this.gateways[cle] = [];
          this.frames[cle] = [];
          axios({
            method: "post",
            url: "http://localhost:3000/devices/" + cle,
            data: {
              ledstatus: false
            }
          }).then(res => {
            console.log("LED Down: " + res.data.message);
          });
        }
      }
      this.drawer = false;
    }
  }
};
</script>
<style>
html,
body,
#app {
  height: 100%;
  margin: 0;
}
</style>
<style scoped>
#app {
  display: flex;
  flex-direction: column;
}
</style>
