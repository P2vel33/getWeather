<template>
  <div
    class="min-w-full min-h-screen max-h-screen overflow-hidden flex justify-center items-center"
  >
    <div
      class="w-full max-w-lg mx-4 p-4 bg-white border border-gray-200 rounded-lg shadow sm:p-8"
    >
      <div v-if="errorMessage">
        <p class="text-base text-center font-normal leading-tight text-red-500">
          {{ errorMessage }}
        </p>
      </div>
      <template v-else>
        <div v-if="loading">Loading</div>
        <div v-else>
          <h5 class="mb-4 text-xl font-medium text-gray-500">
            Погода в <span id="city">{{ weatherData.city }}</span>
          </h5>
          <div class="flex items-baseline text-gray-900">
            <span id="value" class="text-5xl font-extrabold tracking-tight">{{
              weatherData.value
            }}</span>
            <span class="text-5xl font-semibold">°</span>
          </div>
          <ul role="list" class="space-y-5 my-7">
            <li class="flex items-center">
              <span
                class="text-base font-normal leading-tight text-gray-500 ms-3"
              >
                Ощущается как
                <span id="feels_like">{{ weatherData.feelsLike }}</span>
              </span>
            </li>
            <li class="flex">
              <span
                class="text-base font-normal leading-tight text-gray-500 ms-3"
              >
                Влажность
                <span id="humidity">{{ this.weatherData.humidity }}</span
                >%
              </span>
            </li>
            <li class="flex">
              <span
                class="text-base font-normal leading-tight text-gray-500 ms-3"
              >
                Атмосферное давление
                <span id="pressure">{{ this.weatherData.pressure }}</span> мм
                рт. ст.
              </span>
            </li>
          </ul>
        </div>
      </template>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      errorMessage: "",
      loading: true,
      weatherData: {
        city: "",
        value: "",
        feelsLike: "",
        humidity: "",
        pressure: "",
      },
      // API_KEY : import.meta.env.VITE_APP_API_KEY,
      API_KEY : "20dd21edc24ea3b46733e9fc4989774f",
    };
  },
  methods: {
    async getWeatherByCoords(lat, lon) {
      try {
        const response = await fetch(
          `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${this.API_KEY}&lang=ru`
        );
        return await response.json();
      } catch (e) {
          this.errorMessage = "Can't load weather data from the remote host";
      }
    },
    
    async success(pos) {
      const { latitude, longitude } = pos.coords;
      console.log(latitude, longitude);
      const weather = await this.getWeatherByCoords(latitude, longitude);
      if (!weather) {
        this.errorMessage = "Can't load weather data from the remote host";
        this.loading = false;
      } else {
        this.loading = false;
        this.setWeatherData(weather);
      }
    },
    async error(err) {
      if (err.code === 1) {
        console.log("Not enough permissions");
      } else if (err.code === 2) {
        console.log("Position unavailable");
      } else if (err.code === 3) {
        console.log("Timeout exceeded");
      } else {
        console.log("Unknown error");
      }
      const { latitude, longitude } = await this.getLocationByIp();
      this.weatherData = await this.getWeatherByCoords(latitude, longitude);
      if (!this.weatherData) {
        throw new Error("Can't load weather data from the remote host");
      } else {
        setWeatherData(this.weatherData);
      }
    },

    async getLocationByIp() {
      try {
        const response = await fetch("https://ipapi.co/json");
        const { latitude, longitude } = await response.json();
        return { latitude, longitude };
      } catch (e) {
        this.errorMessage = "Can't load weather data from the remote host";
      }
    },

    

    setWeatherData(data) {
      this.weatherData = {
        city: data?.name,
        value: Number(data?.main?.temp - 273.15).toFixed(1),
        feelsLike: Number(data?.main?.feels_like - 273.15).toFixed(1),
        humidity: Number(data?.main?.humidity).toFixed(1),
        pressure: Number(data?.main?.pressure * 0.750062).toFixed(1),
      };
    },
  },
  mounted() {
    navigator.geolocation.getCurrentPosition(this.success, this.error, {
      enableHighAccuracy: true,
      timeout: 5000,
      maximumAge: 0,
    });
  },
};
</script>

<style scoped>

</style>
