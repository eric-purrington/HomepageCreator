<template>
    <div class="inputPage" v-if="!inputRecieved">
        <label for="name">Name to be displayed</label>
        <input type="text" class="name" placeholder="" v-model="info.name"/>
        <label for="zipcode">Zipcode for weather</label>
        <input type="text" class="zipcode" placeholder="98125" v-model="zipcode"/>
        <label for="links">Links on your homepage</label>
        <input type="text" class="links" placeholder="youtube.com"/>


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
            zipcode: '',
            info: {
              name: '',
              date: '',
              time: '',
              weather: {

              },
              links: [

              ],
              background: ''
            }
          }
        },
        methods: {
            // fetchWeather (e) {
            //   fetch("https")
            // },
            // fetchFavicons () {

            // },
            // savePreferences () {
                //save to browser cache and change inputRecieved
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
