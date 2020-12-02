<template>
    <div class="inputPage" v-if="!info.inputRecieved">
        <h1 class="inputHeader">Homepage Creator</h1>

        <!-- INPUT AREA -->
        <label for="name">Display name</label>
        <input type="text" class="name" v-model="info.name"/>

        <label for="zipcode">Zip code for weather</label>
        <input type="text" class="zipcode" v-model="info.zipcode"/>

        <label for="backgroundPref">Background Preference</label>
        <select name="backgroundPref" class="backgroundPref" v-model="info.background">
          <option value="Forest">Forest</option>
          <option value="Mountain">Mountain</option>
          <option value="Beach">Beach</option>
          <option value="Lake">Lake</option>
          <option value="Sky">Sky</option>
          <option value="Random">Random</option>
        </select>

        <!-- SHORTCUT AREA -->
        <div class="shortcutSection">
            <div v-for="shortcut in info.links" :key="shortcut.name" class="shortcutCard">
                <div class="dropdown">
                    <i class="fas fa-ellipsis-v"></i>
                    <div class="dropdownContent">
                      <a href="#" :id="shortcut.name" v-on:click="removeShortcut">Remove</a>
                      <a href="#" :id="shortcut.name" v-on:click="displayEditShortcut">Edit</a>
                    </div>
                </div>
                <a class="shortcutAnchor" :href="shortcut.url">
                    <div>
                        <img :src="shortcut.favicon" :alt="shortcut.name + ' favicon'"/>
                        <h3>{{shortcut.name}}</h3>
                    </div>
                </a>
            </div>
        
            <button v-if="info.links.length !== 10" class="addShortcut" v-on:click="displayShortcutModal"><i class="fas fa-plus fa-3x"></i><br>Add Shortcut</button>
        </div>

        <div id="shortcutModal" class="shortcutModal" ref="shortcutModal">
            <div class="modalContent">
                <h3 v-if="!tempEdit">Add Shortcut</h3>
                <h3 v-if="tempEdit">Edit Shortcut</h3>
                <label for="shortcutName">Name</label>
                <input type="text" class="shortcutName" v-model="tempShortcutName"/>

                <label for="shortcutURL">URL</label>
                <input type="text" class="shortcutURL" placeholder="www.youtube.com" v-model="tempShortcutURL"/>

                <button class="shortcutDone" v-on:click="addOrEditShortcut">Done</button>

                <button class="shortcutCancel" value="shortcutCancel" v-on:click="onClick">Cancel</button>
            </div>
        </div>

        <button class="save" v-on:click="saveAndRender">Save and Update</button>
    </div>


    <div class="displayPage" v-if="info.inputRecieved">
        <h1 class="greeting">Hi, {{info.name}}</h1>

        <div v-for="shortcut in info.links" :key="shortcut.name" class="shortcutCard">
          <!-- <i v-on:click="updateOrDeleteShortcut" :id="shortcut.name" class="fas fa-ellipsis-v"></i> -->
          <a :href="shortcut.url">
            <div>
              <img :src="shortcut.favicon" :alt="shortcut.name + ' favicon'"/>
              <h3>{{shortcut.name}}</h3>
            </div>
          </a>
        </div>

        <button class="edit" v-on:click="switchToEdit">Edit your page</button>
        <p>Photo by {{backgroundPhotographer}} from Pexels</p>

    </div>
</template>

<script>
    import backgrounds from "./util/background.json";

    export default {
        name: 'App',
        data () {
          return {
            date: new Date().toDateString(),
            time: '',
            weather: {},
            possibleBgs: [],
            backgroundIndex: '',
            backgroundPhotographer: '',
            tempEdit: false,
            tempIndexToEdit: 0,
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
            
            if (this.info.background !== "" && this.info.inputRecieved) {
                if (this.info.background == "Random") {
                    this.possibleBgs = backgrounds;
                    this.backgroundIndex = Math.floor(Math.random() * 35);
                } else {
                    this.possibleBgs = backgrounds.filter(background => background.category == this.info.background);
                    this.backgroundIndex = Math.floor(Math.random() * 7);
                }
                this.backgroundPhotographer = this.possibleBgs[this.backgroundIndex].photographer;
                document.body.style.backgroundImage = `linear-gradient(rgba(0, 0, 0, 0.25), rgba(0, 0, 0, 0.25)), url("assets/${this.possibleBgs[this.backgroundIndex].fileName}")`;
            } else {
                document.body.style.background = "linear-gradient(0deg, rgb(0, 0, 0) 10%, rgb(87, 0, 75) 50%, rgb(0, 0, 0, 1) 90%)"
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
        updated: function() {
            if (this.info.background !== "" && this.info.inputRecieved) {
                if (this.info.background == "Random") {
                    this.possibleBgs = backgrounds;
                    this.backgroundIndex = Math.floor(Math.random() * 35);
                } else {
                    this.possibleBgs = backgrounds.filter(background => background.category == this.info.background);
                    this.backgroundIndex = Math.floor(Math.random() * 7);
                }
                this.backgroundPhotographer = this.possibleBgs[this.backgroundIndex].photographer;
                document.body.style.backgroundImage = `linear-gradient(rgba(0, 0, 0, 0.25), rgba(0, 0, 0, 0.25)), url("assets/${this.possibleBgs[this.backgroundIndex].fileName}")`;
            } else {
                document.body.style.background = "linear-gradient(0deg, rgb(0, 0, 0) 10%, rgb(87, 0, 75) 50%, rgb(0, 0, 0, 1) 90%)"
            }
            // if (this.info.zipcode.length === 5 ) {
            //     fetch(`https://api.openweathermap.org/data/2.5/weather?zip=${this.info.zipcode}&appid=${process.env.VUE_APP_WEATHER_API}`)
            //         .then(response => response.json())
            //         .then(data => this.info.weather = data)
            // } 
        },
        methods: {
            saveAndRender: function() {
                this.info.inputRecieved = true
                localStorage.setItem("info", JSON.stringify(this.info));
            },
            switchToEdit: function() {
              this.info.inputRecieved = false
            },
            addOrEditShortcut: function() {
              let fullURL = "https://" + this.tempShortcutURL
              let fetchEndpoint = fullURL.substring(12, fullURL.length);

              fetch(`http://favicongrabber.com/api/grab/${fetchEndpoint}`)
                    .then(response => response.json())
                    .then(data => this.tempShortcutFavicon = data.icons[0].src)
                    .then(() => {
                        if (this.info.links.length === undefined && this.tempEdit === false) {
                            this.info.links = [{"name": this.tempShortcutName, "url": fullURL, "favicon": this.tempShortcutFavicon}]
                        } else if (this.info.links.length !== undefined && this.tempEdit === false){
                            this.info.links.push({"name": this.tempShortcutName, "url": fullURL, "favicon": this.tempShortcutFavicon});
                        } else if (this.tempEdit === true) {
                            this.info.links.splice(this.tempIndexToEdit, 1, {"name": this.tempShortcutName, "url": fullURL, "favicon": this.tempShortcutFavicon});
                        }
                        this.tempEdit = false
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
            displayEditShortcut: function(event) {
                this.tempIndexToEdit = this.info.links.findIndex(i => i.name === event.target.id);
                this.tempEdit = true
                this.displayShortcutModal()
                this.tempShortcutName = event.target.id;
                this.tempShortcutURL = this.info.links[this.tempIndexToEdit].url.substring(8, this.info.links[this.tempIndexToEdit].url.length);
            },
            removeShortcut: function(event) {
                let indexToRemove = this.info.links.findIndex(i => i.name === event.target.id);
                this.info.links.splice(indexToRemove, 1);
            }
        }
    }
</script>

<style>
    html, body {
        height: 100%;
        margin: 0px !important;
        overflow: hidden;
        background-position: center;
        background-repeat: no-repeat;
        background-size: cover;
        position: relative;
    }
    #app {
        font-family: 'Montserrat', sans-serif !important;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: white;
    }

    /* INPUT PAGE */
    .inputHeader {
        font-family: 'Montserrat', sans-serif !important;
        margin: 50px;
        font-size: 40px;
    }

    .name, .zipcode, .backgroundPref {
        margin: 10px auto 30px;
        display: block;
    }

    .backgroundPref option {
      padding: 30px;
    }

    label {
      font-size: 18px;
    }
    /* .backgroundPref {

    } */

    .shortcutSection {
      display: flex;
      margin: 20px auto;
      justify-content: space-evenly;
      flex-wrap: wrap;
      max-width: 50%;
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
    .shortcutCard {
      display: inline-block;
    }
    .dropdown {
      position: relative;
      display: inline-block;
    }
    .dropdownContent {
      display: none;
      position: absolute;
      background-color: #f1f1f1;
      min-width: 160px;
      box-shadow: 0px 8px 16px 0px rgba(0,0,0,0.2);
      z-index: 1;
    }
    .fa-ellipsis-v:hover {
      cursor: pointer;
    }
    .dropdown:hover .dropdownContent{
      display: block;
    }

    /* DISPLAY PAGE */
</style>
