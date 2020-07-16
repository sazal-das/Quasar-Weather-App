<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
        v-model="search"
        @keyup.enter="getWeatherBySearch"
        dark
        borderless
        clearable
        placeholder="Search"
      >
        <template v-slot:before>
          <q-icon name="my_location" @click="getLocation" />
        </template>

        <template v-slot:append>
          <q-btn round dense @click="getWeatherBySearch" flat icon="search" />
        </template>
      </q-input>
    </div>

    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">{{this.weatherData.name}}</div>
        <div class="text-h6 text-weight-light">{{this.weatherData.weather[0].main}}</div>
        <div class="q-my-lg text-h1 text-weight-light relative-position">
          <span>{{ Math.round(this.weatherData.main.temp) }}</span>
          <span class="degree text-h4 relative-position">&deg;C</span>
        </div>
      </div>

      <div class="col text-center">
        <img :src="`http://openweathermap.org/img/wn/${this.weatherData.weather[0].icon}@2x.png`" />
      </div>
    </template>

    <template v-else>
      <div class="col column text-center text-white">
        <div class="text-h2 col text-weight-thin">Weather</div>
        <q-btn class="col" flat @click="getLocation">
          <q-icon left size="3em" name="my_location" />
          <div>Find My Location</div>
        </q-btn>
      </div>
    </template>

    <div class="col skyline"></div>
  </q-page>
</template>

<script>
import Vue from "vue";
import axios from "axios";

export default Vue.extend({
  name: "PageIndex",

  data() {
    return {
      search: "",
      weatherData: null,
      latitude: null,
      longitude: null,
      apiUrl: "https://api.openweathermap.org/data/2.5/weather",
      apiKey: "7e2158e7e859fa0625ae7cc82286721f"
    };
  },

  computed: {
    bgClass() {
      if (this.weatherData) {
        if (this.weatherData.weather[0].icon.endsWith("n")) {
          return "bg-Night";
        } else {
          return "bg-Day";
        }
      }
    }
  },

  methods: {
    getLocation() {
      this.$q.loading.show();
      if (this.$q.platform.is.electron) {
        this.$axios("https://freegeoip.app/json/").then(response => {
          this.latitude = response.data.latitude;
          this.longitude = response.data.longitude;
          this.getWeatherByCoords();
        });
      } else {
        // console.log('Get Location');
        navigator.geolocation.getCurrentPosition(position => {
          // console.log("position", position);
          this.latitude = position.coords.latitude;
          this.longitude = position.coords.longitude;
          // console.log((this.latitude = position.coords.latitude));
          // console.log((this.longitude = position.coords.longitude));
          this.getWeatherByCoords();
        });
      }
    },
    getWeatherByCoords() {
      this.$q.loading.show();
      this.$axios(
        `${this.apiUrl}?lat=${this.latitude}&lon=${this.longitude}&appid=${this.apiKey}&units=metric`
      ).then(response => {
        // console.log('response: ', response);
        this.weatherData = response.data;
        this.$q.loading.hide();
      });
    },
    getWeatherBySearch() {
      this.$q.loading.show();
      this.$axios(
        `${this.apiUrl}?q=${this.search}&appid=${this.apiKey}&units=metric`
      ).then(response => {
        // console.log('response: ', response);
        this.weatherData = response.data;
        this.$q.loading.hide();
      });
    }
  }
});
</script>

<style lang="scss">
.q-page {
  background: linear-gradient(to right, #136a8a, #267871);
  &.bg-Night {
    background: linear-gradient(to right, #232526, #414345);
  }
  &.bg-Day {
    background: linear-gradient(to right, #2193b0, #6dd5ed);
  }
}
.degree {
  top: -44px;
}
.skyline {
  flex: 0 0 100px;
  background: url("../assets/images/skyline.png");
  background-size: contain;
  background-position: center bottom;
}
</style>
