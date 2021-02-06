<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-ot-lg q-px-md">
      <q-input
        v-model="search"
        @keyup.enter="getWeatherBySearch"
        placeholder="Search"
        dark
        borderless
      >
        <template v-slot:before>
          <q-icon name="my_location" @click="getLocation" />
        </template>

        <template v-slot:append>
          <q-btn round dense flat @click="getWeatherBySearch" icon="search" />
        </template>
      </q-input>
    </div>

    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-white-light">
          {{ weatherData.name }}
        </div>
        <div class="text-h6 text-white-light">
          {{ weatherData.weather[0].main }}
        </div>
        <div class="text-h1 text-white-thin q-my-lg relative-position">
          <span>
            {{ Math.round(weatherData.main.temp) }}
          </span>
          <span class="text-h3 relative-position degree"> &deg;C </span>
        </div>
      </div>

      <div class="col text-center">
        <img
          :src="`https://www.openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`"
        />
      </div>
    </template>

    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">
          Quasar <br />
          Weather
        </div>

        <q-btn class="col" flat @click="getLocation">
          <q-icon left size="3em" name="my_location" />
          <div>Find my location</div>
        </q-btn>
      </div>
    </template>

    <div class="col skyline">
      <img src="" alt="" />
    </div>
  </q-page>
</template>

<script>
export default {
  name: "PageIndex",
  data() {
    return {
      search: "",
      weatherData: false,
      lat: null,
      lon: null,
      apiUrl: "http://api.openweathermap.org/data/2.5/weather",
      apiKey: "e273a3bdfe84ae78fc43cc3a54cf6681",
    };
  },
  computed: {
    bgClass() {
      if (this.weatherData) {
        if (this.weatherData.weather[0].icon.endsWith('n')) {
          return 'bg-night'
        }
        else {
          return 'bg-day'
        }
      }
    }
  },
  methods: {
    getLocation() {
      this.$q.loading.show()
      if(this.$q.platform.is.electron) {
        this.$axios.get('https://freegeoip.app/json/').then(response => {
          this.lat = response.data.latitude;
          this.lon = response.data.longitude;
          this.getWeatherByCoords();
        })
      }
      else {
        navigator.geolocation.getCurrentPosition((position) => {
          console.log("position: ", position);
          this.lat = position.coords.latitude;
          this.lon = position.coords.longitude;
          this.getWeatherByCoords();
        });
      }
    },
    getWeatherByCoords() {
      this.$q.loading.show()
      this.$axios(
        `${this.apiUrl}?lat=${this.lat}&lon=${this.lon}&appid=${this.apiKey}&units=metric`
      ).then((response) => {
        this.weatherData = response.data;
        this.$q.loading.hide()
      });
    },
    getWeatherBySearch() {
      this.$q.loading.show()
      this.$axios(
        `${this.apiUrl}?q=${ this.search }&appid=${this.apiKey}&units=metric`
      ).then((response) => {
        this.weatherData = response.data;
        this.$q.loading.hide()
      });
    },
  },
};
</script>

<style lang="sass">
.q-page
  background: linear-gradient(to bottom, #136a8a, #267871)
  &.bg-night
    background: linear-gradient(to bottom, #232526, #414345)
  &.bg-day
    background: linear-gradient(to bottom, #00b4db, #0083b0)
.degree
  top: -34px
  left: -25px
.skyline
  flex: 0 0 100px
  background: url(../statics/skyline.png)
  background-size: contain
  background-position: center bottom
</style>
