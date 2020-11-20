<template>
    <div class="inputPage" v-if="!inputRecieved">
        <label for="name">Name to be displayed</label>
        <input type="text" class="name" placeholder="" v-model="info.name"/>
        <label for="zipcode">Zipcode for weather</label>
        <input type="text" class="zipcode" placeholder="98125" v-model="info.zipcode"/>
        <label for="links">Links on your homepage</label>
        <input type="text" class="links" placeholder="youtube.com"/>
    {{info.date}}
     {{info.time}}

    </div>
    <DisplayPage info="info" :render="inputRecieved"/>
</template>

<script>
    import DisplayPage from './components/DisplayPage.vue'

    export default {
        name: 'App',
        components: {
            DisplayPage
        },
        data () {
          return {
            inputRecieved: false,
            info: {
              name: '',
              zipcode: '',
              date: new Date().toDateString(),
              time: ' ',
              weather: {

              },
              links: [

              ],
              linkFavicons: [

              ],
              background: ''
            } || JSON.parse(localStorage.getItem("usersInfo"))
          }
        },
        mounted : function() {           
            setInterval(() => {
                this.info.time = new Date().toLocaleTimeString();
            }, 1000);
        },
        methods: {
            // fetchWeatherFaviconsAndSave: () => {
            //     fetch(`https://api.openweathermap.org/data/2.5/weather?zip=${this.zipcode}&appid=${process.env.VUE_APP_WEATHER_API}`)
            //       .then(response => response.json())
            //       .then(data => this.info.weather = data)
            //     fetch(`http://favicongrabber.com/api/grab/${this.info.link}`)
            //       .then(response => response.json())
            //       .then(data => this.info.linkFavicons = data.icons[0].src)
            //     localStorage.setItem("usersInfo", JSON.stringify(this.info));
            // }
        }
    }
</script>

<style>
    #app {
        font-family: Avenir, Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
        margin-top: 60px;
    }
    label, input {
      display: flex;
      margin: 30px;
    }
</style>
