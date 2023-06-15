<template>
  <div>
    <nav class="navbar navbar-light bg-light">
    <h1>Google Map Search Demo</h1>
    <div class="d-flex">
    <form @submit.prevent="searchLocation" class="d-flex" >
        <input class="form-control me-1" type="text" v-model="address" placeholder="search the location">
        <button type="submit"  class="btn btn-outline-success my-2 my-sm-0 me-2">Search</button>
    </form>
    <button @click="setCurrentLocation" class="btn btn-outline-success me-2">Acquire User's Current Location</button>
    </div> 
    </nav>

    <h4 v-if="latestedTimeZone" class="time">Time zone for the latest searched location is: "{{latestedTimeZone}}"</h4>
    <h4 v-if="latestedLocalTime" class="time">Local time for the lasest searched location is:  "{{latestedLocalTime.year}}/{{latestedLocalTime.month}}/{{latestedLocalTime.day}} {{latestedLocalTime.hours}}hr(s)/{{latestedLocalTime.minutes}}min(s)/{{latestedLocalTime.seconds}}s"</h4>

    <hr>

    <HistoryTable :searchedHistory="searchedHistory" :key="searchedHistory.length"/>
    <button @click="deleteSelectedLocations" class="btn btn-outline-success my-2 my-sm-0">Delete Selected Searched Address(es) </button>
    <hr>
    <div class="d-flex justify-content-center">
    <div id="map"></div>
    </div>
  </div>
</template>

<script>
import { Loader } from '@googlemaps/js-api-loader';
import HistoryTable from './HistoryTable.vue'
export default {
    components:{HistoryTable},
    data(){
        return {
            apiKey: process.env.VUE_APP_GOOGLE_MAP_API, 
            Marker: null,
            map:null,
            loader:null,
            geocoder: null,
            mapOptions:null,
            address: null, 
            searchedHistory: [],
            latestedTimeZone: null, 
            latestedLocalTime: null,
        }
    },
    mounted(){
            this.loader = new Loader({
            apiKey: this.apiKey,
            version: "weekly",
            libraries: ["places"],
            });

            this.mapOptions = {
            mapId: "DEMO_MAP_ID",
            center: {lat: 43.6532, lng: -79.3832},
            zoom: 4
            };
            
            this.loader.load().then(
                async() =>{
                    this.map = new google.maps.Map(document.getElementById("map"), this.mapOptions);
                    this.geocoder = new google.maps.Geocoder()
                    const {Marker} = await google.maps.importLibrary("marker")
                    this.Marker = Marker;
                }
            )
    },
    methods: {
        setCurrentLocation: function(){
            if (navigator.geolocation){
                navigator.geolocation.getCurrentPosition(
                    (position)=>{
                        const currentPos = {
                            lat: position.coords.latitude, 
                            lng: position.coords.longitude
                        }
                        this.address = `${currentPos.lat}, ${currentPos.lng}`
                        this.searchLocation()
                    }, ()=>{
                        alert("error fetching the location data")
                    }
                )
            } else {
                alert("error fetching location")
            }
        },
        searchLocation: function(){
            this.geocoder.geocode({'address': this.address}, (results, status)=>{
                if (status == 'OK'){
                    this.fetchTime(results[0].geometry.location.lat(), results[0].geometry.location.lng())
                    this.setMarker(results)
                } else {
                     alert('Geocode was not successful for the following reason: ' + status);
                }
            })
        }, 
        setMarker: function(results){
                let historyItem = results[0]
                historyItem.marker = new this.Marker({
                    title: historyItem.place_id, 
                    position: results[0].geometry.location,
                    map: this.map
                })
                historyItem.selection=false //for the use of cancellation 
                this.searchedHistory.push(historyItem)
                this.map.setCenter(results[0].geometry.location)
        },
        fetchTime: function(lat, lng){
            const time = Math.round(Date.now()/1000)
            const url = `https://maps.googleapis.com/maps/api/timezone/json?location=${lat}%2C${lng}&timestamp=${time}&key=${this.apiKey}`
            fetch(url)
                .then(res=>res.json()).then(data=>{
                    if(data.status == 'OK'){
                        this.latestedTimeZone = data.timeZoneName
                        const calculatedTime = new Date((time + data.dstOffset + data.rawOffset) * 1000);
                        this.latestedLocalTime = {}
                        this.latestedLocalTime.year = calculatedTime.getFullYear()
                        this.latestedLocalTime.month = calculatedTime.getMonth() + 1
                        this.latestedLocalTime.day = calculatedTime.getDay()
                        this.latestedLocalTime.hours = calculatedTime.getHours()
                        this.latestedLocalTime.minutes = calculatedTime.getMinutes()
                        this.latestedLocalTime.seconds = calculatedTime.getSeconds()
                    }
                })
                .catch(error=>{alert("error" + error)})
        },
        deleteSelectedLocations: function(){
            if (this.searchedHistory.length > 0){
                this.searchedHistory = this.searchedHistory.filter((item)=>{
                    if (item.selection === false){
                        return true
                    } else {
                        item.marker.setVisible(false)
                        return false
                    }   
                })
            }
        }
    }}
</script>

<style>
    #map{
    width:90%;
    height:500px;
    background-color: grey;
    }
    .time{
        text-align: left;
        margin-left: 5px;
    }
</style>