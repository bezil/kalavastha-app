<template>
  <q-page class="flex column" :class="bgclass">
    <div class="col q-pt-lg q-px-md">
      <q-input v-model="search" placeholder="Search" dark borderless @keyup.enter="getWeatherbyName">
        <template v-slot:before>
          <q-icon @click="getCurrentLocation" name="my_location" />
        </template>
        <template v-slot:append>
          <q-btn round dense flat icon="search" @click="getWeatherbyName"/>
        </template>
      </q-input>
    </div>
    <template v-if="weatherdata">
        <div class="col text-white text-center">
          <div class="text-h4 text-weight-light">
              {{weatherdata.name}}
          </div>
          <div class="text-h6 text-weight-light">
            {{weatherdata.weather[0].main}}
          </div>
          <div class="text-h1 text-weight-thin q-my-lg relative-position">
            <span>{{Math.round(weatherdata.main.temp)}}</span> <span class="text-h4 relative-position degree">&deg;C</span>
          </div>
          <div class="text-center">
            <img :src="`https://openweathermap.org/img/wn/${ weatherdata.weather[0].icon }@2x.png`" />
          </div>
        </div>
    </template>
    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h4 text-weight-thin">
          Kalavastha <br>Weather App
        </div>

           <q-btn @click="getCurrentLocation" class="col" flat>
              <q-icon left size="3em" name="my_location" />
              <div class="text-weight-light">Find my Location</div>
           </q-btn>

      </div>
    </template>


    <div class="col citybg"></div>
  </q-page>
</template>

<script>
export default {
  name: 'PageIndex',
  data(){
      return {
        search: '',
        weatherdata: null,
        lat: null,
        lon: null,
        apikey: '7b782885bf10a4d0e311b4e146375f4c',
        apiURL: 'https://api.openweathermap.org/data/2.5/weather',
      }
  },
  computed:{
    bgclass(){
      if(this.weatherdata){
        if(this.weatherdata.weather[0].icon.endsWith('n')){return 'bgnight';}
        else{return 'bgday';}
      }
    }
  },
  methods: {
    getCurrentLocation(){
      this.$q.loading.show();
      if(this.$q.platform.is.electron || this.$q.platform.is.cordova){
          this.$axios.get('https://freegeoip.app/json/').then(response => {
               this.lat = response.data.latitude;
               this.lon = response.data.longitude;
               this.getWeatherbyCoords();
               console.log('freeip app data');
          })
      }
      else{
          navigator.geolocation.getCurrentPosition(position => {
                                            console.log('navigator');
                                            this.lat = position.coords.latitude;
                                            this.lon = position.coords.longitude;
                                            this.getWeatherbyCoords();
                                          });
      }


    },
    getWeatherbyCoords() {
       this.$q.loading.show();
            this.$axios(`${this.apiURL}?lat=${this.lat}&lon=${this.lon}&appid=${this.apikey}&units=metric`)
            .then(response => {
               this.weatherdata = response.data;
               this.$q.loading.hide();
            })
    },
     getWeatherbyName() {
        this.$q.loading.show();
            this.$axios(`${this.apiURL}?q=${this.search}&appid=${this.apikey}&units=metric`)
            .then(response => {
                  this.weatherdata = response.data;
                  this.$q.loading.hide();
            }).catch((error) => {
                 console.warn('Not good man :(');
                  this.$q.loading.hide();
            })
    }
  }
}
</script>

<style lang="scss">
  .q-page{
      background: #2C3E50;  /* fallback for old browsers */
      background: -webkit-linear-gradient(to top, #4CA1AF, #2C3E50);  /* Chrome 10-25, Safari 5.1-6 */
      background: linear-gradient(to top, #4CA1AF, #2C3E50); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */
  }
  .bgnight{
      background: #0F2027;  /* fallback for old browsers */
      background: -webkit-linear-gradient(to bottom, #2C5364, #203A43, #0F2027);  /* Chrome 10-25, Safari 5.1-6 */
      background: linear-gradient(to bottom, #2C5364, #203A43, #0F2027); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */
  }
  .bgday{
      background: #457fca;  /* fallback for old browsers */
      background: -webkit-linear-gradient(to top, #5691c8, #457fca);  /* Chrome 10-25, Safari 5.1-6 */
      background: linear-gradient(to top, #5691c8, #457fca); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */
  }
  .degree{
    top:-44px;
  }
  .citybg{
    flex: 0 0 50px;
    background: url(../assets/city.png);
    background-size: contain;
    background-position: center bottom;
  }
</style>
