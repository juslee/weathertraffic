<template>
  <div class="weather_traffic">
    <div class="date_time">
      <span>Date:</span>
      <span>
        <input 
          type="date" 
          v-model="date"
        >
      </span>
      <span>Time:</span>
      <span>
        <input 
          type="time" 
          v-model="time"
        >
      </span>
    </div>
    <div class="traffic">
      <span>Traffic Locations:</span>
      <select v-model="selected">
        <option
          disabled 
          :value="{ name: '', image: '' }"
        >
          Please select one
        </option>
        <option
          v-for="location in locations"
          :key="location.name"
          :value="{ name: location.name, image: location.image }"
        >
          {{ location.name }}
        </option>
      </select>
      <span class="forecast">
        Forecast: {{ forecast }}
      </span>
    </div>
    <div>
      <img 
        id="image"
        :src="selected.image"
        :alt="selected.name"
      >
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'WeatherTraffic',
  data() {
    return {
      date: new Date(new Date().getTime()-(new Date().getTimezoneOffset()*60*1000)).toISOString().slice(0,10),
      time: new Date().toLocaleTimeString(),
      locations: [],
      selected: { name: '', image: '' },
      forecast: 'No available forecast',
    }
  },
  created() {
    this.getLocations()
  },
  computed: {
    date_time: {
      get() {
        return this.date + 'T' + encodeURIComponent(this.time)
      },
    }
  },
  watch: {
    date() {
      this.getLocations()
    },
    time() {
      this.getLocations()
    },
    selected() {
      this.getWeather()
    }
  },
  methods: {
    async getLocations () {
      try {
        const traffic_images = `https://api.data.gov.sg/v1/transport/traffic-images?date_time=${this.date_time}`
        const response = await axios.get(traffic_images)
        this.locations = await Promise.all(response.data.items[0].cameras.map(async (camera) => { 
          const reverse_geocoding = `https://api.bigdatacloud.net/data/reverse-geocode-client?latitude=${camera.location.latitude}&longitude=${camera.location.longitude}`
          const response = await axios.get(reverse_geocoding)
          return {
            geocoding: camera.locations,
            name: response.data.locality,
            image: camera.image,
          }
        }))
      } catch (error) {
        console.log(error)
      }
    },
    async getWeather () {
      const weather = `https://api.data.gov.sg/v1/environment/2-hour-weather-forecast?date_time=${this.date_time}`
      const response = await axios.get(weather)
      const index = response.data.items[0].forecasts.findIndex(forecast => this.selected.name.startsWith(forecast.area))
      this.forecast = index === -1 ? 'No available forecast' : response.data.items[0].forecasts[index].forecast
    },
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#image {
  width: 320px;
  height: auto;
}

@media screen and (max-width: 480px) {
  .forecast {
    display: block;
  }
}
</style>
