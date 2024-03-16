<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
        v-model="search"
        @keyup.enter="getWeatherBySearch"
        placeholder="Пошук"
        dark
        borderless
      >
        <template v-slot:before>
          <q-icon @click="getLocation" name="my_location" />
        </template>

        <template v-slot:append>
          <q-btn @click="getWeatherBySearch" round dense flat icon="search" />
        </template>
      </q-input>
    </div>

    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">{{ weatherData.name }}</div>
        <div class="text-h6 text-weight-light">
          {{ weatherData.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{ Math.round(weatherData.main.temp) }}</span>
          <span class="text-h4 relative-position degree">&deg;C</span>
        </div>
      </div>

      <div class="col text-center">
        <img
          :src="`https://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`"
          alt=""
        />
      </div>
    </template>

    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">Quasar<br />Weather</div>
      </div>
      <q-btn @click="getLocation" class="col" flat>
        <q-icon left size="3em" name="my_location" />
        <div>Пошук моєї геолокації</div>
      </q-btn>
    </template>

    <div class="col skyline"></div>
  </q-page>
  <!-- Нижній бар (footer) -->
  <q-footer
    class="bg-grey text-white text-center d-flex justify-center align-center"
    style="height: 80px"
  >
    <div>Odnosum Denys(121-21-1)</div>
  </q-footer>
</template>

<script>
import axios from "axios";

export default {
  name: "IndexPage",
  data() {
    return {
      search: "",
      weatherData: null,
      lat: null,
      lon: null,
      apiUrl: "https://api.openweathermap.org/data/2.5/weather",
      apiKey: "1f05f988851e4bced654347b75cd89f1",
    };
  },
  computed: {
    // eslint-disable-next-line vue/return-in-computed-property
    bgClass() {
      if (this.weatherData) {
        if (this.weatherData.weather[0].icon.endsWith("n")) {
          return "bg-night";
        } else {
          return "bg-day";
        }
      }
    },
  },
  methods: {
    getLocation() {
      this.$q.loading.show();

      if (this.$q.platform.is.electron) {
        axios.get("https://api.ipbase.com/v1/json/").then((response) => {
          this.lat = response.data.latitude;
          this.lon = response.data.longitude;
          this.getWeatherByCoords();
        });
      } else {
        navigator.geolocation.getCurrentPosition((position) => {
          console.log("position: ", position);
          this.lat = position.coords.latitude;
          this.lon = position.coords.longitude;
          this.getWeatherByCoords();
        });
      }
    },
    getWeatherByCoords() {
      this.$q.loading.show();
      axios
        .get(
          `${this.apiUrl}?lat=${this.lat}&lon=${this.lon}&appid=${this.apiKey}&units=metric`
        )
        .then((response) => {
          this.weatherData = response.data;
          this.$q.loading.hide();
        })
        .catch((error) => {
          console.error("error: ", error);
        });
    },
    getWeatherBySearch() {
      this.$q.loading.show();
      axios
        .get(
          `${this.apiUrl}?q=${this.search}&appid=${this.apiKey}&units=metric`
        )
        .then((response) => {
          this.weatherData = response.data;
          this.$q.loading.hide();
        })
        .catch((error) => {
          console.error("error: ", error);
        });
    },
  },
};
</script>

<style lang="sass">
.q-page
  background: linear-gradient(to top, #159957, #155799)
  &.bg-night
    background: linear-gradient(to right, #000000, #434343)
  &.bg-day
    background: linear-gradient(to right, #00d2ff, #928dab)
.degree
  top: -44px
.skyline
  flex: 0 0 100px
  background: url(../icons/skyline.png)
  background-size: contain
  background-position: center bottom
.bg-grey
  background-color: grey
.d-flex
  display: flex
.justify-center
  justify-content: center
.align-center
  align-items: center
</style>
