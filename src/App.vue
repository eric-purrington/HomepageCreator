<template>
    <div class="inputPage" v-if="!info.inputRecieved">
        <h1>Welcome to the homepage creator</h1>
        <h2>Input your info here</h2>
        <label for="name">Name to be displayed</label>
        <input type="text" class="name" placeholder="" v-model="info.name"/>

        <label for="zipcode">Zipcode for weather</label>
        <input type="text" class="zipcode" placeholder="98125" v-model="info.zipcode"/>

        <h2>Input your pref here</h2>
        <label for="backgroundPref">Background Preference</label>
        <select name="backgroundPref" id="backgroundPref" v-model="info.background">
          <option value="Forest">Forest</option>
          <option value="Mountain">Mountain</option>
          <option value="Beach">Beach</option>
        </select>
        <label for="addShortcut">Add Shortcut</label>
        <button name="addShortcut" class="addShortcut" v-on:click="addShortcut"><i class="fas fa-plus fa-3x"></i></button>
        <button class="save" v-on:click="fetchFaviconsAndSave">Save and Render</button>
    </div>
    <div class="displayPage" v-if="info.inputRecieved">
        <h1 class="greeting">Hi, {{info.name}}</h1>
        <button class="edit" v-on:click="switchToEdit">Edit your page</button>

    </div>
    <!-- <DisplayPage info="info" :render="inputRecieved"/> -->
</template>

<script>
    // import DisplayPage from './components/DisplayPage.vue'

    export default {
        name: 'App',
        // components: {
        //     DisplayPage
        // },
        data () {
          return {
            date: new Date().toDateString(),
            time: '',
            weather: "",
            info: {
              inputRecieved: false,
              name: '',
              zipcode: '',
              links: {},
              background: ''
            }
          }
        },
        mounted : function() {
            setInterval(() => {
                this.time = new Date().toLocaleTimeString();
            }, 1000);

            if (JSON.parse(localStorage.getItem("info"))) {
              this.info = JSON.parse(localStorage.getItem("info"))
            }
            // if (this.info.zipcode.length === 5 ) {
            //     fetch(`https://api.openweathermap.org/data/2.5/weather?zip=${this.info.zipcode}&appid=${process.env.VUE_APP_WEATHER_API}`)
            //         .then(response => response.json())
            //         .then(data => this.info.weather = data)
            // } 
        },
        // updated: () => {
            // if (this.info.zipcode.length === 5 ) {
            //     fetch(`https://api.openweathermap.org/data/2.5/weather?zip=${this.info.zipcode}&appid=${process.env.VUE_APP_WEATHER_API}`)
            //         .then(response => response.json())
            //         .then(data => this.info.weather = data)
            // } 
        // },
        methods: {
            fetchFaviconsAndSave: function() {
                // fetch(`http://favicongrabber.com/api/grab/${this.info.link}`)
                //     .then(response => response.json())
                //     .then(data => this.info.linkFavicons = data.icons[0].src)
                this.info.inputRecieved = true
                localStorage.setItem("info", JSON.stringify(this.info));
            },
            switchToEdit: function() {
              this.info.inputRecieved = false
            }
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
