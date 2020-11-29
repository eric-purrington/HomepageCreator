<template>
    <div class="inputPage" v-if="!info.inputRecieved">
        <h1 class="inputHeader">Homepage Creator</h1>

        <label for="name">Display name</label>
        <input type="text" class="name" v-model="info.name"/>

        <label for="zipcode">Zip code for weather</label>
        <input type="text" class="zipcode" v-model="info.zipcode"/>

        <label for="backgroundPref">Background Preference</label>
        <select name="backgroundPref" id="backgroundPref" v-model="info.background">
          <option value="Forest">Forest</option>
          <option value="Mountain">Mountain</option>
          <option value="Beach">Beach</option>
          <option value="Lake">Lake</option>
        </select>

        <div v-for="shortcut in info.links" :key="shortcut.name" class="shortcutCard">
          <i v-on:click="updateOrDeleteShortcut" :id="shortcut.name" class="fas fa-ellipsis-v"></i>
          <a :href="shortcut.url">
            <div>
              <img :src="shortcut.favicon" :alt="shortcut.name + ' favicon'"/>
              <h3>{{shortcut.name}}</h3>
            </div>
          </a>
        </div>
        
        <button v-if="info.links.length !== 10" class="addShortcut" v-on:click="displayShortcutModal"><i class="fas fa-plus fa-3x"></i><br>Add Shortcut</button>

        <div id="shortcutModal" class="shortcutModal" ref="shortcutModal">
            <div class="modalContent">
                <h3>Add Shortcut</h3>
                <label for="shortcutName">Name</label>
                <input type="text" class="shortcutName" v-model="tempShortcutName"/>

                <label for="shortcutURL">Full URL</label>
                <input type="text" class="shortcutURL" placeholder="www.youtube.com" v-model="tempShortcutURL"/>
                <button class="shortcutDone" v-on:click="addShortcut">Done</button>
                <button class="shortcutCancel" value="shortcutCancel" v-on:click="onClick">Cancel</button>
            </div>
        </div>

        <button class="save" v-on:click="fetchFaviconsAndSave">Save and Render</button>
        {{info}}
    </div>
    <div class="displayPage" v-if="info.inputRecieved">
        <h1 class="greeting">Hi, {{info.name}}</h1>

        <div v-for="shortcut in info.links" :key="shortcut.name" class="shortcutCard">
          <i v-on:click="updateOrDeleteShortcut" :id="shortcut.name" class="fas fa-ellipsis-v"></i>
          <a :href="shortcut.url">
            <div>
              <img :src="shortcut.favicon" :alt="shortcut.name + ' favicon'"/>
              <h3>{{shortcut.name}}</h3>
            </div>
          </a>
        </div>

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
            weather: {},
            tempShortcutName: '',
            tempShortcutURL: '',
            tempShortcutFavicon: '',
            info: {
              inputRecieved: false,
              name: '',
              zipcode: '',
              links: [],
              background: ''
            }
          }
        },
        mounted: function() {
            setInterval(() => {
                this.time = new Date().toLocaleTimeString();
            }, 1000);

            if (JSON.parse(localStorage.getItem("info"))) {
              this.info = JSON.parse(localStorage.getItem("info"))
            }

            window.addEventListener('click', this.onClick);
            // if (this.info.zipcode.length === 5 ) {
            //     fetch(`https://api.openweathermap.org/data/2.5/weather?zip=${this.info.zipcode}&appid=${process.env.VUE_APP_WEATHER_API}`)
            //         .then(response => response.json())
            //         .then(data => this.info.weather = data)
            // } 
        },
        beforeUnmount: function() {
            window.removeEventListener('click', this.onClick);
        },
        // updated: function() {
            // if (this.info.zipcode.length === 5 ) {
            //     fetch(`https://api.openweathermap.org/data/2.5/weather?zip=${this.info.zipcode}&appid=${process.env.VUE_APP_WEATHER_API}`)
            //         .then(response => response.json())
            //         .then(data => this.info.weather = data)
            // } 
        // },
        methods: {
            fetchFaviconsAndSave: function() {
                this.info.inputRecieved = true
                localStorage.setItem("info", JSON.stringify(this.info));
            },
            switchToEdit: function() {
              this.info.inputRecieved = false
            },
            addShortcut: function() {
              let fullURL = "https://" + this.tempShortcutURL
              let fetchEndpoint = fullURL.substring(12, fullURL.length);

              fetch(`http://favicongrabber.com/api/grab/${fetchEndpoint}`)
                    .then(response => response.json())
                    .then(data => this.tempShortcutFavicon = data.icons[0].src)
                    .then(() => {
                        if (this.info.links.length === undefined) {
                            this.info.links = [{"name": this.tempShortcutName, "url": fullURL, "favicon": this.tempShortcutFavicon}]
                        } else {
                            this.info.links.push({"name": this.tempShortcutName, "url": fullURL, "favicon": this.tempShortcutFavicon});
                        }
                        this.$refs["shortcutModal"].style.display = "none";
                    })
            },
            onClick: function(event) {
              if(event.target == this.$refs["shortcutModal"] || event.target.value == "shortcutCancel") {
                this.$refs["shortcutModal"].style.display = "none";
              }
            },
            displayShortcutModal: function() {
                this.tempShortcutName = "";
                this.tempShortcutURL = "";
                this.tempShortcutFavicon = "";
                this.$refs["shortcutModal"].style.display = "block";
            },
            updateOrDeleteShortcut: function() {

            }
        }
    }
</script>

<style>
    html {
      /* background: linear-gradient(90deg, rgb(0, 0, 0) 0%, rgb(52, 0, 87) 50%, rgb(0, 0, 0, 1) 100%); */
      height: 100%;
      background: radial-gradient(rgb(0, 0, 0), rgb(129, 0, 133), rgb(0, 0, 0, 1));
    }
    #app {
        font-family: 'Montserrat', sans-serif !important;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: white;
    }
    .inputHeader {
        font-family: 'Montserrat', sans-serif !important;
        margin: 50px;
        font-size: 40px;
    }
    .shortcutModal {
        display: none; /* Hidden by default */
        position: fixed; /* Stay in place */
        z-index: 1; /* Sit on top */
        padding-top: 100px; /* Location of the box */
        left: 0;
        top: 0;
        width: 100%; /* Full width */
        height: 100%; /* Full height */
        overflow: auto; /* Enable scroll if needed */
        background-color: rgb(0,0,0); /* Fallback color */
        background-color: rgba(0,0,0,0.75); /* Black w/ opacity */
    }
    .modalContent {
        background-color: white;
        color: black;
        margin: auto;
        padding: 20px;
        border-radius: 25px;
        width: 50%;
    }
</style>
