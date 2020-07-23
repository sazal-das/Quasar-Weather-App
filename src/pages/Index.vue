<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-px-md">
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
        <div class="text-bold">{{this.day}}</div>
        <div class="text-bold">{{this.date}}</div>
        <div class="text-bold q-pb-sm">{{this.time}}</div>
        <div class="text-h6 text-weight-light">{{this.weatherData.name}}</div>
        <div class="text-bold text-weight-light">{{this.weatherData.weather[0].main}}</div>
        <div class="text-h3 text-weight-light relative-position">
          <span>{{ Math.round(this.weatherData.main.temp) }}</span>
          <span class="degree text-h5 relative-position">&deg;C</span>
        </div>
      </div>

      <div class="col text-center">
        <img :src="`http://openweathermap.org/img/wn/${this.weatherData.weather[0].icon}@2x.png`" />
      </div>
    </template>

    <template v-else>
      <div class="col text-center text-white">
        <div class="text-h2 col text-weight-thin q-pa-lg">Weather</div>
        <q-btn class="col q-pa-lg" flat @click="getLocation">
          <q-icon left size="3em" name="my_location" />
          <div>Find My Location</div>
        </q-btn>
      </div>
    </template>

    <template v-if="this.weatherData">
      <div class="row q-pl-md q-pr-md text-center">
        <div class="col text-white text-bold">Sunrise</div>
        <div class="col text-white text-bold">Sunset</div>
        <div class="col text-white text-bold">Wind</div>
        <div class="col text-white text-bold">Humidity</div>
      </div>
      <div class="row q-pl-md q-pr-md text-center">
        <div class="col">
          <img :src="require('../assets/images/sunrise.png')" alt width="50px" height="50px" />
        </div>
        <div class="col">
          <img :src="require('../assets/images/sunsets.png')" alt width="50px" height="50px" />
        </div>
        <div class="col">
          <img :src="require('../assets/images/wind.png')" alt width="50px" height="50px" />
        </div>
        <div class="col">
          <img :src="require('../assets/images/humidity.png')" alt width="50px" height="50px" />
        </div>
      </div>
      <div class="row q-pl-md q-pr-md text-center">
        <div class="col text-white text-bold">{{this.sunrise}}</div>
        <div class="col text-white text-bold">{{this.sunset}}</div>
        <div class="col text-white text-bold">{{this.weatherData.main.humidity}}</div>
        <div class="col text-white text-bold">{{this.weatherData.wind.speed}}</div>
      </div>
    </template>
    <template v-if="this.weatherData">
      <div class="row text-center q-pl-md q-pr-md">
        <div class="col text-bold text-white">{{this.dayName1}}</div>
        <div class="col text-bold text-white">{{this.dayName2}}</div>
        <div class="col text-bold text-white">{{this.dayName3}}</div>
        <div class="col text-bold text-white">{{this.dayName4}}</div>
        <div class="col text-bold text-white">{{this.dayName5}}</div>
      </div>
      <div class="row text-center q-pl-md q-pr-md">
        <div class="col"><img class="col icon" :src="`http://openweathermap.org/img/wn/${this.daysData.list[0].weather[0].icon}@2x.png`" /></div>
        <div class="col"><img class="icon" :src="`http://openweathermap.org/img/wn/${this.daysData.list[6].weather[0].icon}@2x.png`" /></div>
        <div class="col"><img class="icon" :src="`http://openweathermap.org/img/wn/${this.daysData.list[14].weather[0].icon}@2x.png`" /></div>
        <div class="col"><img class="icon" :src="`http://openweathermap.org/img/wn/${this.daysData.list[22].weather[0].icon}@2x.png`" /></div>
        <div class="col"><img class="icon" :src="`http://openweathermap.org/img/wn/${this.daysData.list[30].weather[0].icon}@2x.png`" /></div>
      </div>
      <div class="row text-center q-pl-md q-pr-md">
        <div class="col">
          <span class="text-bold text-white">{{ Math.round(this.daysData.list[6].main.temp) }}</span>
          <span class="degree2 text-white relative-position">&deg;C</span>
        </div>
        <div class="col">
          <span class="text-bold text-white">{{ Math.round(this.daysData.list[14].main.temp) }}</span>
          <span class="degree2 text-white relative-position">&deg;C</span>
        </div>
        <div class="col">
          <span class="text-bold text-white">{{ Math.round(this.daysData.list[22].main.temp) }}</span>
          <span class="degree2 text-white relative-position">&deg;C</span>
        </div>
        <div class="col">
          <span class="text-bold text-white">{{ Math.round(this.daysData.list[30].main.temp) }}</span>
          <span class="degree2 text-white relative-position">&deg;C</span>
        </div>
        <div class="col">
          <span class="text-bold text-white">{{ Math.round(this.daysData.list[37].main.temp) }}</span>
          <span class="degree2 text-white relative-position">&deg;C</span>
        </div>
      </div>
    </template>

    <div class="col skyline"></div>
  </q-page>
</template>

<script>
const moment = require("moment");
import Vue from "vue";
import axios from "axios";
import { log } from "util";

export default Vue.extend({
  name: "PageIndex",

  data() {
    return {
      search: "",
      weatherData: null,
      daysData: null,
      latitude: null,
      longitude: null,
      apiUrl: "https://api.openweathermap.org/data/2.5/weather",
      apiUrl2: "https://api.openweathermap.org/data/2.5/forecast",
      apiKey: "7e2158e7e859fa0625ae7cc82286721f",
      sunrise: null,
      sunset: null,
      time: "",
      date: "",
      day: "",
      rain: false,
      thunderstorm: false,
      dayName1: null,
      dayName2: null,
      dayName3: null,
      dayName4: null,
      dayName5: null,

    };
  },

  computed: {
    bgClass() {
      if (this.weatherData) {
        // if (this.weatherData.weather[0].icon.endsWith("n")) {
        //   return "bg-Night";
        // } else {
        //   return "bg-Day";
        // }
        if (this.weatherData.weather[0].main === "Thunderstorm") {
          return "thunderstorm";
        } else if (this.weatherData.weather[0].main === "Rain") {
          return "rain";
        } else if (this.weatherData.weather[0].main === "Clouds") {
          return "clouds";
        } else if (this.weatherData.weather[0].main === "Clear") {
          if (this.weatherData.weather[0].icon.endsWith("n")) {
            return "night_clear";
          } else {
            return "clear";
          }
        } else if (this.weatherData.weather[0].main === "Haze") {
          if (this.weatherData.weather[0].icon.endsWith("n")) {
            return "bg-Night";
          } else {
            return "bg-Day";
          }
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
        console.log(this.weatherData);

        let sRise = new Date(this.weatherData.sys.sunrise * 1000);
        if (sRise.getHours() % 12 >= 1) {
          this.sunrise = `${sRise.getHours() % 12}:${sRise.getMinutes()} AM`;
        } else {
          this.sunrise = `${sRise.getHours()}:${sRise.getMinutes()} AM`;
        }
        console.log(`${sRise.getHours()}:${sRise.getMinutes()} AM`);

        let sSet = new Date(this.weatherData.sys.sunset * 1000);
        if (sSet.getHours() % 12 >= 1) {
          this.sunset = `${sSet.getHours() % 12}:${sSet.getMinutes()} PM`;
        } else {
          this.sunset = `${sSet.getHours()}:${sSet.getMinutes()} PM`;
        }

        let time = new Date().getTime();
        this.date = moment(time).format("LL");
        this.time = moment(time).format("LT");
        this.day = moment(time).format("dddd");

        this.$q.loading.hide();
      });
      // Get 5 days weather
      this.$axios(
        `${this.apiUrl2}?lat=${this.latitude}&lon=${this.longitude}&appid=${this.apiKey}&units=metric`
      ).then(response => {
        // console.log('response: ', response);
        this.daysData = response.data;
        console.log(this.daysData);
        let alldays = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'];

        let a = new Date(this.daysData.list[6].dt * 1000);
        this.dayName1 = alldays[a.getDay()];
        
        let b = new Date(this.daysData.list[14].dt * 1000);
        this.dayName2 = alldays[b.getDay()];

        let c = new Date(this.daysData.list[22].dt * 1000);
        this.dayName3 = alldays[c.getDay()];

        let d = new Date(this.daysData.list[30].dt * 1000);
        this.dayName4 = alldays[d.getDay()];

        let e = new Date(this.daysData.list[37].dt * 1000);
        this.dayName5 = alldays[e.getDay()];
        
        
      });
    },


    getWeatherBySearch() {
      this.$q.loading.show();
      this.$axios(
        `${this.apiUrl}?q=${this.search}&appid=${this.apiKey}&units=metric`
      ).then(response => {
        // console.log('response: ', response);
        this.weatherData = response.data;

        let sRise = new Date(this.weatherData.sys.sunrise * 1000);
        if (sRise.getHours() % 12 >= 1) {
          this.sunrise = `${sRise.getHours() % 12}:${sRise.getMinutes()} AM`;
        } else {
          this.sunrise = `${sRise.getHours()}:${sRise.getMinutes()} AM`;
        }
        console.log(`${sRise.getHours()}:${sRise.getMinutes()} AM`);

        let sSet = new Date(this.weatherData.sys.sunset * 1000);
        if (sSet.getHours() % 12 >= 1) {
          this.sunset = `${sSet.getHours() % 12}:${sSet.getMinutes()} PM`;
        } else {
          this.sunset = `${sSet.getHours()}:${sSet.getMinutes()} PM`;
        }

        let time = new Date().getTime();
        this.date = moment(time).format("LL");
        this.time = moment(time).format("LT");
        this.day = moment(time).format("dddd");

        this.$q.loading.hide();
      });
      // Get 5 days weather
      this.$axios(
        `${this.apiUrl2}?q=${this.search}&appid=${this.apiKey}&units=metric`
      ).then(response => {
        // console.log('response: ', response);
        this.daysData = response.data;
        console.log(this.daysData);
        let alldays = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'];

        let a = new Date(this.daysData.list[6].dt * 1000);
        this.dayName1 = alldays[a.getDay()];
        
        let b = new Date(this.daysData.list[14].dt * 1000);
        this.dayName2 = alldays[b.getDay()];

        let c = new Date(this.daysData.list[22].dt * 1000);
        this.dayName3 = alldays[c.getDay()];

        let d = new Date(this.daysData.list[30].dt * 1000);
        this.dayName4 = alldays[d.getDay()];

        let e = new Date(this.daysData.list[37].dt * 1000);
        this.dayName5 = alldays[e.getDay()];
        
        
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
  &.thunderstorm {
    background: url("../assets/images/flash2.jpg");
  }
  &.rain {
    background: url("../assets/images/rain.png");
    background-size: cover;
  }
  &.clouds {
    background: url("../assets/images/clouds.jpg");
  }
  &.clear {
    background: url("../assets/images/clear.jpg");
  }
  &.night_clear {
    background: url("../assets/images/night_clear.png");
  }
}
.degree {
  top: -23px;
}
.degree2 {
  top: -5px;
}
.skyline {
  flex: 0 0 100px;
  background: url("../assets/images/skyline.png");
  background-size: contain;
  background-position: center bottom;
}
.icon{
  height: auto;
  width: 50px;
}
</style>
