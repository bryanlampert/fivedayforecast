<template>
  <div class="container">
    <div class="search">
      <input
        type="text"
        v-model.lazy="city"
        v-on:keyup.enter.prevent="getWeather"
        placeholder="Enter a city"
        class="searchBar rounded"
      />
    </div>
    <h2 class="city">{{city.toUpperCase()}}</h2>
    <div v-if="errorMsg" class="text-danger">{{errorMsg}}</div>
    <div v-if="loading">
      <i class="fas fa-spinner fa-pulse"></i>
    </div>
    <div v-if="city && !errorMsg">
      <div class="container border rounded">
        <div class="row align-items-center forecast">
          <div v-if="weather.today" class="col p-1 border">
            <div v-text="date.today" class="date" />
            <h2 class="iconContainer">
              <i :class="weatherIcon(weather.today.weather[0].id)"></i>
            </h2>
            <span class="temp">
              <span v-text="weather.today.main.temp | 0"></span> &deg;C
            </span>
            <div v-text="weather.today.weather[0].description" class="description" />
          </div>

          <div v-if="weather.tomorrow" class="col p-1 border">
            <div v-text="date.tomorrow" class="date" />
            <h2 class="iconContainer">
              <i :class="weatherIcon(weather.tomorrow.weather[0].id)"></i>
            </h2>
            <span class="temp">
              <span v-text="weather.tomorrow.main.temp | 0"></span> &deg;C
            </span>
            <div v-text="weather.tomorrow.weather[0].description" class="description" />
          </div>

          <div v-if="weather.dayThree" class="col p-1 border">
            <div v-text="date.dayThree" class="date" />
            <h2 class="iconContainer">
              <i :class="weatherIcon(weather.dayThree.weather[0].id)"></i>
            </h2>
            <span class="temp">
              <span v-text="weather.dayThree.main.temp | 0"></span> &deg;C
            </span>
            <div v-text="weather.dayThree.weather[0].description" class="description" />
          </div>

          <div v-if="weather.dayFour" class="col p-1 border">
            <div v-text="date.dayFour" class="date" />
            <h2 class="iconContainer">
              <i :class="weatherIcon(weather.dayFour.weather[0].id)"></i>
            </h2>
            <span class="temp">
              <span v-text="weather.dayFour.main.temp | 0"></span> &deg;C
            </span>
            <div v-text="weather.dayFour.weather[0].description" class="description" />
          </div>

          <div v-if="weather.dayFive" class="col p-1 border">
            <div v-text="date.dayFive" class="date" />
            <h2 class="iconContainer">
              <i :class="weatherIcon(weather.dayFive.weather[0].id)"></i>
            </h2>
            <span class="temp">
              <span v-text="weather.dayFive.main.temp | 0"></span> &deg;C
            </span>
            <div v-text="weather.dayFive.weather[0].description" class="description" />
          </div>
        </div>
      </div>
      <div class="pressure">
        Weekly Average Pressure: {{ avgPressure }} hPa
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
let moment = require('moment');

moment().format();

Date.prototype.addDays = function(days) {
  var d = new Date(this.valueOf());
  d.setDate(d.getDate() + days);
  return d;
}
let day = new Date();
let today = day.toDateString();
let tomorrow = day.addDays(1).toDateString();
let dayThree = day.addDays(2).toDateString();
let dayFour = day.addDays(3).toDateString();
let dayFive = day.addDays(4).toDateString();

export default {
  name: 'Weather',
  data () {
    return {
      city: '',
      weather: {
        today: '',
        tomorrow: '',
        dayThree: '',
        dayFour: '',
        dayFive: ''
      },
      date: {
        today: today,
        tomorrow: tomorrow,
        dayThree: dayThree,
        dayFour: dayFour,
        dayFive: dayFive
      },
      avgPressure: '',
      apiKey: '79e012cedaf18f5c7c0b43a1322e045b',
      loading: false,
      errorMsg: ''
    }
  },
  methods: {
    getWeather: function() {
      this.loading = true;
      axios.get(`https://api.openweathermap.org/data/2.5/forecast?units=metric&lang=en&&APPID=${this.apiKey}&q=${this.city}
      `).then(response => {
        this.errorMsg = '';
        this.weather.today = response.data.list[0];
        let futureDateWeather = [];
        let weekPressureSum = 0;
        let range = response.data.cnt;
        // Loop through response data
        for (let i = 0; i < range; i++) {
          let dt = moment.unix(response.data.list[i].dt);
          let dat = moment(dt).get('hour');
          if (dat > 10 && dat < 14) {
            // push weather data for middle of day
            futureDateWeather.push(response.data.list[i]);
          }
          // sum of weekly pressure to calculate average
          weekPressureSum += response.data.list[i].main.pressure;
        }
        // set weather data
        this.weather.tomorrow = futureDateWeather[1];
        this.weather.dayThree = futureDateWeather[2];
        this.weather.dayFour = futureDateWeather[3];
        this.weather.dayFive = futureDateWeather[4];
        // set average weekly pressure
        this.avgPressure = Math.round(weekPressureSum / range);
        this.loading = false;
      }).catch(error => {
        this.city = '';
        this.errorMsg = "Please enter a valid city!";
        this.loading = false;
      });
    },
    weatherIcon (id) {
      // set the weather icon based on the openweathermap api weather codes
      if (id >= 200 && id < 233) {
        return 'wi wi-thunderstorm';
      } else if (id >= 300 && id < 322) {
        return 'wi wi-sprinkle';
      } else if (id >= 500 && id < 532) {
        return 'wi wi-rain-mix';
      } else if (id >= 600 && id < 623) {
        return 'wi wi-snow-wind';
      } else if (id >= 701 && id < 772) {
        return 'wi wi-smoke';
      } else if(id == 781 || id == 900) {
        return 'wi wi-tornado';
      } else if(id == 800) {
        return 'wi wi-day-sunny';
      } else if (id >= 801 && id < 805) {
        return 'wi wi-cloudy';
      } else if (id == 901 || id == 960 || id == 961) {
        return 'wi wi-storm-warning';
      } else if(id == 902 || id == 962) {
        return 'wi wi-hurricane-warning';
      } else if(id == 903) {
        return 'wi wi-thermometer-exterior';
      } else if(id == 904) {
        return 'wi wi-thermometer';
      } else if (id == 905 || (id >= 951 && id < 960)) {
        return 'wi wi-strong-wind';
      } else if(id == 906) {
        return 'wi wi-hail';
      }
    }
  },
}
</script>
