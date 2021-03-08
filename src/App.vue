<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
      <v-icon x-large>mdi-earth</v-icon>
      <span class="text-h5 ml-2">Countries of the World</span>
      <v-spacer></v-spacer>

      <v-btn large href="https://github.com/blkhwks19/countries" target="_blank">
        <v-icon class="mr-2">mdi-github</v-icon>
        Github
      </v-btn>

      <!-- TABS -->
      <template v-slot:extension>
        <v-tabs
          v-model="tab"
          fixed-tabs
        >
          <v-tab
            v-for="(region, index) in regions"
            :key="index"
          >
            {{ region }}
          </v-tab>
        </v-tabs>
      </template>
    </v-app-bar>


    <v-main>

      <!-- TAB ITEMS -->
      <v-tabs-items v-model="tab">
        <v-tab-item
          v-for="(region, index) in regions"
          :key="index"
        >
          <!-- COUNTRY LIST BY REGION -->
          <Region :region="region" @showDialog="showCountry($event)" />
        </v-tab-item>
      </v-tabs-items>

      <!-- BACK TO TOP BUTTON -->
      <v-fab-transition>
        <v-btn
          fab
          color="primary"
          dark
          fixed
          bottom
          right
          v-show="showScrollBtn"
          v-scroll="onScroll"
          @click="scrollToTop()"
        >
          <v-icon>mdi-chevron-up</v-icon>
        </v-btn>
      </v-fab-transition>

      <!-- COUNTRY DETAILS DIALOG -->
      <div class="text-center">
        <v-dialog
          v-model="showDialog"
          width="450"
          transition="dialog-top-transition"
        >
          <v-card>
            <!-- MAP -->
            <v-sheet width="450" height="300">
              <MglMap 
                :accessToken="accessToken" 
                :mapStyle.sync="mapStyle" 
                :center="center" 
                :zoom.sync="zoom"
              >
                <!-- Nav control buttons are missing labels ??? -->
                <!-- <MglNavigationControl position="top-right"/> -->
              </MglMap>
            </v-sheet>

            <!-- <v-img 
              :src="country.flag"
              style="border-bottom: 1px solid rgba(0, 0, 0, 0.12)"
            ></v-img> -->

            <v-card-title>
              {{ country.name }}
              <v-tooltip top open-delay="500">
                <template v-slot:activator="{ on }">
                  <v-btn 
                    icon 
                    x-small 
                    color="primary" 
                    class="ml-2" 
                    :href="`https://en.wikipedia.org/wiki/${country.name}`"
                    target="_blank"
                    v-on="on"
                  >
                    <v-icon>mdi-open-in-new</v-icon>
                  </v-btn>
                </template>
                <span>More Info</span>
              </v-tooltip>

              <v-spacer></v-spacer>

              <v-img 
                :src="country.flag"
                style="border: 1px solid rgba(0, 0, 0, 0.12); top:5px;"
                max-width="47"
              ></v-img>
            </v-card-title>
            <v-card-subtitle class="pb-2">{{ country.subregion }}</v-card-subtitle>

            <v-divider></v-divider>

            <v-card-subtitle class="pt-2 pb-0"><span class="font-weight-bold">Population:</span> {{ country.population.toLocaleString() }}</v-card-subtitle>
            <v-card-subtitle class="pb-0"><span class="font-weight-bold">Area:</span> {{ getArea(country.area) }} mi<sup>2</sup></v-card-subtitle>
            <v-card-subtitle class="pb-2"><span class="font-weight-bold">Capital:</span> {{ country.capital }}</v-card-subtitle>

            <v-divider></v-divider>

            <v-card-text class="py-1">
              <v-card-subtitle class="px-0 py-0 font-weight-medium">Languages</v-card-subtitle>
              <v-chip-group column>
                <v-chip
                  v-for="(lang, index) in country.languages"
                  :key="index"
                  @click.prevent=""
                  :ripple="false"
                  color="primary"
                  small
                >
                  {{ lang.name }}
                </v-chip>
              </v-chip-group>
            </v-card-text>

            <v-divider></v-divider>

            <v-card-text class="py-1">
              <v-card-subtitle class="px-0 py-0 font-weight-medium">Currencies</v-card-subtitle>
              <v-chip-group column>
                <v-chip
                  v-for="(curr, index) in country.currencies"
                  :key="index"
                  @click.prevent=""
                  :ripple="false"
                  color="primary"
                  small
                >
                  {{ curr.name }}
                </v-chip>
              </v-chip-group>
            </v-card-text>

            <v-divider></v-divider>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn text color="primary" @click="showDialog = false">close</v-btn>
            </v-card-actions>

          </v-card>
        </v-dialog>
      </div>

    </v-main>
  </v-app>
</template>

<script>
import Region from '@/components/Region';
import Mapbox from "mapbox-gl";
import { MglMap, MglNavigationControl } from "vue-mapbox";

export default {
  name: 'App',

  components: {
    Region,
    MglMap,
    MglNavigationControl,
  },

  data() {
    return {
      tab: null,
      regions: ['Africa', 'Americas', 'Asia', 'Europe', 'Oceania'],
      showScrollBtn: false,
      showDialog: false,
      country: { population:0 },
      showDetails: false,
      showLang: false,
      showCurr: false,
      accessToken: 'pk.eyJ1IjoiYmxraHdrczE5IiwiYSI6ImNrbHpldTJvMjFib2sydXA4bjluOXltNWcifQ.FphWr-2ysYbcTKbDQ4sCSA',
      mapStyle: 'mapbox://styles/mapbox/streets-v11',
      center: [0, 0],
      zoom: 3,
    }
  },

  created() {
    // We need to set mapbox-gl library here in order to use it in template
    this.mapbox = Mapbox;
  },

  methods: {
    onScroll(e) {
      if (typeof window === 'undefined') return;
      const top = window.pageYOffset || e.target.scrollTop || 0;
      this.showScrollBtn = top > 20;
    },

    scrollToTop() {
      this.$vuetify.goTo(0);
    },

    showCountry(country) {
      this.country = country;
      this.center = [this.country.latlng[1], this.country.latlng[0]];
      this.zoom = 3;
      this.showDialog = true;
    },

    getArea(sqkm) {
      return Math.round(sqkm / 2.59).toLocaleString();
    },
  }
};
</script>
