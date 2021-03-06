<template>
    <div class="displayPage">
        <!-- Greeting depending on time of day -->
        <h1 v-if="timeEasy >= 5 && timeEasy < 12" class="greeting">Good Morning, {{info.name}}</h1>
        <h1 v-if="timeEasy >= 12 && timeEasy < 18" class="greeting">Good Afternoon, {{info.name}}</h1>
        <h1 v-if="timeEasy >= 18 || timeEasy < 5" class="greeting">Good Evening, {{info.name}}</h1>
        <h2 class="date">{{date}}</h2>
        <h2 class="time">{{time}}</h2>
        
        <!-- Weather depending on postal code given -->
        <a v-if="hasWeather" href="https://www.weather.com" class="weatherLink" ><div class="weatherCard">
          <p>{{weather.weather[0].description[0].toUpperCase()}}{{weather.weather[0].description.substring(1, weather.weather[0].description.length)}}</p>
          <p>{{weather.main.temp}}&deg; {{weather.name}}</p>
        </div></a>

        <!-- Shortcut cards displaying name given and favicon if retrievable -->
        <div class="shortcutSection">
            <div v-for="shortcut in info.links" :key="shortcut.name" class="shortcutCard">
                <div class="dropdown">
                    <i class="fas fa-ellipsis-v"></i>
                    <div class="dropdownContent">
                      <a class="removeShorty" href="#" :id="shortcut.name" v-on:click="removeShortcut">Remove</a>
                      <a class="editShorty" href="#" :id="shortcut.name" v-on:click="displayEditShortcut">Edit</a>
                    </div>
                </div>
                <a class="shortcutAnchor" :href="shortcut.url">
                    <div>
                        <img v-if="shortcut.favicon != ''" class="shortcutIcon" :src="shortcut.favicon" :alt="shortcut.name + ' favicon'"/>
                        <img v-if="shortcut.favicon == ''" class="shortcutIcon" src="https://via.placeholder.com/900/000000/FFFFFF/?text=Icon" :alt="shortcut.name + ' favicon placeholder'"/>
                        <p v-if="shortcut.name.length < 10" class="shortcutName">{{shortcut.name}}</p>
                        <p v-if="shortcut.name.length >= 10" class="shortcutName">{{shortcut.name.substring(0, 9)}}...</p>
                    </div>
                </a>
            </div>
            <button class="shortcutCard" v-if="info.links.length !== 10" id="addShortcut" v-on:click="displayShortcutModal"><i class="fas fa-plus fa-3x"></i><br>Add Shortcut</button>
        </div>

        <!-- Shortcut modal for editing or adding a shortcut -->
        <div v-if="openShortcutModal" id="shortcutModal" class="shortcutModal" ref="shortcutModal">
            <div class="modalContent">
                <h3 v-if="!tempEdit">Add Shortcut</h3>
                <h3 v-if="tempEdit">Edit Shortcut</h3>
                <label for="shortcutModName">Name</label>
                <input type="text" class="shortcutModName" v-model="tempShortcutName"/>

                <label for="shortcutURL">Full URL</label>
                <input type="text" class="shortcutURL" placeholder="https://www.youtube.com" v-model="tempShortcutURL"/>
                <p v-if="showUrlError" class="errorMsg">Input valid URL</p>
                <p v-if="showTryAgain" class="errorMsg">Whoops! Give it another go.</p>

                <button class="shortcutDone" v-on:click="addOrEditShortcut">Done</button>

                <button class="shortcutCancel" value="shortcutCancel" v-on:click="onClick">Cancel</button>
            </div>
        </div>

        <!-- Input modal -->
        <div v-if="!info.inputRecieved" id="customizeModal" class="customizeModal" ref="customizeModal">
            <div class="customizeModalContent">
                <h3>Customize</h3>
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

                <button class="shortcutDone" v-on:click="closePrefModal">Done</button>
            </div>
        </div>

        <!-- Customize button -->
        <button class="edit" v-on:click="switchToEdit"><i class="fas fa-pen"></i> Customize</button>

        <!-- Photo Cred -->
        <p class="photoCred">Photo by {{backgroundPhotographer}} from Pexels</p>
        
        <!-- Tab counter for fun -->
        <div class="topRight">
            <p class="tabCount">{{tabCount}} tabs created!</p>
            <a class="gmail" href="https://mail.google.com">Gmail</a>
            <a class="drive" href="https://drive.google.com">Drive</a>
        </div>
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
            timeEasy: '',
            weather: {},
            hasWeather: false,
            possibleBgs: [],
            backgroundIndex: '',
            backgroundPhotographer: '',
            tempEdit: false,
            tempIndexToEdit: 0,
            tempShortcutName: '',
            tempShortcutURL: '',
            tempShortcutFavicon: '',
            showUrlError: false,
            openShortcutModal: false,
            showTryAgain: false,
            tabCount: 0,
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
            // Tab counter and saver
            if (!JSON.parse(localStorage.getItem("tabCount"))) {
                this.tabCount = 1;
                JSON.stringify(localStorage.setItem("tabCount", this.tabCount))
            } else {
                this.tabCount = JSON.parse(localStorage.getItem("tabCount"));
                this.tabCount++;
                JSON.stringify(localStorage.setItem("tabCount", this.tabCount))
            }
            
            // Keeps time up to... time
            setInterval(() => {
                this.time = new Date().toLocaleTimeString('en-US', {
                    hour: '2-digit',
                    minute: '2-digit'
                });

                if (this.time[0] == "0") {
                    this.time = this.time.substring(1, this.time.length)
                }

                this.timeEasy = new Date().toLocaleTimeString('en-US', {
                    hour12: false, 
                    hour: '2-digit'
                });
                if (this.timeEasy[0] == "0") {
                    this.timeEasy = parseInt(this.timeEasy.substring(1, this.timeEasy.length))
                }
            }, 1000);

            // Retrieves data from localstorage
            if (JSON.parse(localStorage.getItem("info"))) {
                this.info = JSON.parse(localStorage.getItem("info"))
            }
            
            this.styleBackground();
            this.fetchWeather(); 

            // For closing modals
            window.addEventListener('click', this.onClick);
        },
        beforeUnmount: function() {
            window.removeEventListener('click', this.onClick);
        },
        methods: {
            // Opens input modal
            switchToEdit: function() {
                this.info.inputRecieved = false;
            },
            // Adds or Edits shortcut given modal input
            addOrEditShortcut: function() {
                this.showUrlError = false;
                this.showTryAgain = false;
                let urlObj;
                let fetchEndpoint;

                // Checks to see if URL is real
                try {
                    urlObj = new URL(this.tempShortcutURL);
                } catch(e) {
                    console.log(e);
                }

                // If URL real initiate fetching favicon else display error
                if (urlObj != undefined) {
                    if (urlObj.hostname.startsWith("www.")) {
                        fetchEndpoint = urlObj.hostname.substring(4, urlObj.hostname.length)
                    } else {
                        fetchEndpoint = urlObj.hostname;
                    }

                  // Fetches favicon and saves the links data to info
                    fetch(`https://favicongrabber.com/api/grab/${fetchEndpoint}`)
                        .then(response => response.json())
                        .then(data => this.tempShortcutFavicon = data.icons[0].src)
                        .then(() => {
                            if (this.info.links.length === undefined && this.tempEdit === false) {
                                this.info.links = [{
                                "name": this.tempShortcutName, 
                                "url": this.tempShortcutURL, 
                                "favicon": this.tempShortcutFavicon}]
                            } else if (this.info.links.length !== undefined && this.tempEdit === false){
                                this.info.links.push({
                                "name": this.tempShortcutName, 
                                "url": this.tempShortcutURL, 
                                "favicon": this.tempShortcutFavicon});
                            } else if (this.tempEdit === true) {
                              this.info.links.splice(this.tempIndexToEdit, 1, {
                                "name": this.tempShortcutName, 
                                "url": this.tempShortcutURL, 
                                "favicon": this.tempShortcutFavicon});
                            }
                            this.tempEdit = false;
                            this.openShortcutModal = false;
                            localStorage.setItem("info", JSON.stringify(this.info))})
                        .catch((e) => {
                            if (e instanceof SyntaxError) {
                                this.showTryAgain = true;
                            } else if (e instanceof TypeError) {
                                if (this.info.links.length === undefined && this.tempEdit === false) {
                                    this.info.links = [{
                                    "name": this.tempShortcutName, 
                                    "url": this.tempShortcutURL, 
                                    "favicon": ""}]
                                } else if (this.info.links.length !== undefined && this.tempEdit === false){
                                    this.info.links.push({"name": this.tempShortcutName, 
                                    "url": this.tempShortcutURL, 
                                    "favicon": ""});
                                } else if (this.tempEdit === true) {
                                    this.info.links.splice(this.tempIndexToEdit, 1, {
                                    "name": this.tempShortcutName, 
                                    "url": this.tempShortcutURL, 
                                    "favicon": ""});
                                }
                                this.tempEdit = false;
                                this.openShortcutModal = false;
                                localStorage.setItem("info", JSON.stringify(this.info))
                            }
                        });
                } else {
                    this.showUrlError = true;
                }
            },
            // Closes modal on outside click
            onClick: function(event) {
                if(event.target == this.$refs["shortcutModal"] || event.target.value == "shortcutCancel") {
                    this.openShortcutModal = false;
                }
            },
            displayShortcutModal: function() {
                this.tempShortcutName = "";
                this.tempShortcutURL = "";
                this.tempShortcutFavicon = "";
                this.openShortcutModal = true;
            },
            displayEditShortcut: function(event) {
                this.openShortcutModal = true;
                this.tempIndexToEdit = this.info.links.findIndex(i => i.name === event.target.id);
                this.tempEdit = true;
                this.displayShortcutModal();
                this.tempShortcutName = event.target.id;
                this.tempShortcutURL = this.info.links[this.tempIndexToEdit].url;
            },
            removeShortcut: function(event) {
                let indexToRemove = this.info.links.findIndex(i => i.name === event.target.id);
                this.info.links.splice(indexToRemove, 1);
                localStorage.setItem("info", JSON.stringify(this.info));
            },
            // Randomly selects background from JSON if preference given else gives standard purple gradient
            styleBackground: function() {
                if (this.info.background !== "" && this.info.inputRecieved) {
                    if (this.info.background == "Random") {
                        this.possibleBgs = backgrounds;
                        this.backgroundIndex = Math.floor(Math.random() * 35);
                    } else {
                        this.possibleBgs = backgrounds.filter(background => background.category == this.info.background);
                        this.backgroundIndex = Math.floor(Math.random() * 7);
                    }
                    this.backgroundPhotographer = this.possibleBgs[this.backgroundIndex].photographer;
                    document.body.style.backgroundImage = `linear-gradient(rgba(0, 0, 0, 0.5), rgba(0, 0, 0, 0.5)), url("assets/${this.possibleBgs[this.backgroundIndex].fileName}")`;
                } else {
                    document.body.style.background = "linear-gradient(0deg, rgb(0, 0, 0) 10%, rgb(87, 0, 75) 50%, rgb(0, 0, 0, 1) 90%)"
                }
            },
            // Fetches Weather from openweathermap
            fetchWeather: function() {
                if (this.info.zipcode.length === 5 ) {
                    fetch(`https://api.openweathermap.org/data/2.5/weather?zip=${this.info.zipcode}&units=imperial&appid=${process.env.VUE_APP_WEATHER_API}`)
                        .then(response => response.json())
                        .then(data => {
                            this.weather = data;
                            this.hasWeather = true;
                        });  
                } else {
                    this.hasWeather = false;
                }
            },
            closePrefModal: function() {
                this.info.inputRecieved = true;
                this.styleBackground();
                this.fetchWeather();
                localStorage.setItem("info", JSON.stringify(this.info));
            }
        }
    }
</script>

<style>
    html, body {
        height: 100%;
        margin: 0px !important;
        overflow: hidden;
        background-position: center !important;
        background-repeat: no-repeat !important;
        background-size: cover !important;
        position: relative;
    }

    #app {
        font-family: 'Montserrat', sans-serif !important;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: white;
    }

    .name, .zipcode, .backgroundPref, .shortcutModName, .shortcutURL {
        margin: 10px auto 30px;
        display: block;
    }

    .backgroundPref {
        margin-bottom: 30px;
    }

    .shortcutSection {
        display: flex;
        margin: 125px auto 25px;
        justify-content: space-evenly;
        flex-wrap: wrap;
        max-width: 50%;
    }
    .shortcutModal, .customizeModal {
        display: block;
        position: fixed;
        z-index: 1;
        padding-top: 100px;
        left: 0;
        top: 0;
        width: 100%;
        height: 100%;
        background-color: rgb(0,0,0);
        background-color: rgba(0,0,0,0.75);
    }
    .modalContent, .customizeModalContent {
        background-color: white;
        color: black;
        margin: auto;
        padding: 10px;
        border-radius: 25px;
        width: 25%;
    }

    .modalContent h3, .customizeModalContent h3 {
        font-family: 'Montserrat', sans-serif !important;
        color: rgb(51, 51, 51);
    }
    .shortcutDone, .shortcutCancel {
        background-color: rgb(51, 51, 51);
        color: white;
        margin: 10px;
        border: none;
        border-radius: 5px;
        padding: 10px;
    }
    .shortcutModName, .shortcutURL {
        width: 50%;
    }
    .shortcutCard {
        background-color: #ffffff00;
        padding: 10px;
        border-radius: 20px;
        margin-top: 10px;
        width: 120px;
        height: auto;
    }
    .shortcutCard:hover {
        background-color: rgba(255, 255, 255, 0.25);
    }
    .shortcutIcon {
        height: 30px;
        background-color: rgb(51, 51, 51);
        padding: 7.5px;
        border-radius: 20px;
        margin: 5px;
    }
    .shortcutName, .fa-plus {
        margin: 5px;
    }
    .save {
        border: none;
        border-radius: 20px;
        font-size: 20px;
        padding: 20px;
        font-family: 'Montserrat', sans-serif !important;
        color: rgb(51, 51, 51);
        margin: 10px;
        background-color: white;
    }
    .dropdown {
        position: relative;
        display: inline-block;
        left: 40%;
        bottom: -15%;
        color: white;
    }
    .dropdownContent {
        visibility: hidden;
        position: absolute;
        color: white;
        background-color: rgb(51, 51, 51);
        min-width: 100px;
        box-shadow: 0px 8px 16px 0px rgba(0,0,0,0.2);
        z-index: 1;
        border-radius: 5px;
        padding: 5px;
        transition: .35s;
    }
    .removeShorty, .editShorty {
        margin: 10px;
        color: white;
        display: block;
    }
    .errorMsg {
        color: red;
    }
    #addShortcut {
        border: none;
        border-radius: 20px;
        color: white;
        background-color: rgb(51, 51, 51);
        padding: 10px;
        box-shadow: 0px 8px 16px 0px rgba(0,0,0,0.25);
    }
    #addShortcut:hover, .edit:hover {
        color: rgb(51, 51, 51);
        background-color: white;
    }
    .removeShorty:visited, .editShorty:visited, .shortcutAnchor, .shortcutAnchor:visited, .gmail:visited, .drive:visited {
        color: white;
        text-decoration: none;
    }
    #addShortcut:hover, .save:hover, .edit:hover, .shortcutDone:hover, .shortcutCancel:hover {
        cursor: pointer;
    }
    .dropdown:hover .dropdownContent {
        transition-delay: 0s;
        visibility: visible;
    }
    .edit {
        position: absolute;
        bottom: 5%;
        right: 2.5%;
        color: white;
        background-color: rgb(51, 51, 51);
        padding: 7px;
        border: none;
        border-radius: 10px;
        font-style: oblique;
        box-shadow: 0px 8px 16px 0px rgba(0,0,0,0.25);
    }
    .edit i {
        margin: 0px 5px;
    }
    .photoCred{
        position: absolute;
        bottom: 5%;
        left: 2.5%;
    }
    .topRight {
        position: absolute;
        top: 2.5%;
        right: 2.5%;
    }
    .gmail, .drive {
        color: white;
        margin: 10px;
    }
    .weatherCard {
        position: absolute;
        top: 2.5%;
        left: 2.5%;
        color:white;
        background-color: rgb(51, 51, 51);
        padding: 10px;
        border-radius: 20px;
    }
    .weatherCard:hover {
      background-color:white;
      color: rgb(51, 51, 51);
    }
    .greeting {
      font-size: 48px;
      margin: 50px;
    }
    .date, .time {
      font-size: 24px;
    }
    .weatherCard p {
      font-size: 16px;
      margin: 8px !important;
    }
</style>
