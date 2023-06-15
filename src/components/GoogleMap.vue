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

    <h3 v-if="latestedTimeZone">Time zone for the latest searched location is "{{latestedTimeZone}}"</h3>
    <h3 v-if="latestedLocalTime">Local time for the lasest searched location is "{{latestedLocalTime.year}}/{{latestedLocalTime.month}}/{{latestedLocalTime.day}}  {{latestedLocalTime.hours}}hr(s)/{{latestedLocalTime.minutes}}min(s)/{{latestedLocalTime.seconds}}s"</h3>
    <hr>

    <HistoryTable :searchedHistory="searchedHistory" :key="searchedHistory.length"/>
    <button @click="deleteSelectedLocations" class="btn btn-outline-success my-2 my-sm-0">Delete Selected Searched Address(es) </button>
    <hr>
    <div id="map"></div>
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
            AdvancedMarkerElement: null,
            map:null,
            loader:null,
            geocoder: null,
            mapOptions:null,
            marker:null, 
            currentPos: null,
            address: null, 
            searchedHistory: [],
            latestedTimeZone: null, 
            latestedLocalTime: null,
            tempSelectedIDList: []
        }
    },
    mounted(){
        this.initMap()
            //   this.marker = new google.maps.Marker({

            //             position: {lat:43.6532, lng: -79.3832},
            //             map: this.map
            //             })
    },
    methods: {
        initMap: function(){
            this.loader = new Loader({
            apiKey: this.apiKey,
            version: "weekly",
            libraries: ["places"],
            });

            this.mapOptions = {
            mapId: "DEMO_MAP_ID",
            center: {
                lat: 43.6532,
                lng: -79.3832
            },
            zoom: 4
            };
            
            this.loader.importLibrary("maps").then(({Map})=>{
                this.map = new Map(document.getElementById("map"), this.mapOptions);
                this.geocoder = new google.maps.Geocoder()
            })
            this.loader.importLibrary("marker")
            this.loader.importLibrary("marker").then(({AdvancedMarkerElement})=>{
                this.AdvancedMarkerElement = AdvancedMarkerElement
                const marker = new AdvancedMarkerElement({
                    map:this.map
                })
                }
                 
            )
        },
        setCurrentLocation: function(){
            console.log("set")
            if(navigator.geolocation){
                navigator.geolocation.getCurrentPosition(
                    (position)=>{
                        this.currentPos = {
                            lat: position.coords.latitude, 
                            lng: position.coords.longitude
                        }
                        console.log(this.currentPos)
                        this.address = `${this.currentPos.lat}, ${this.currentPos.lng}`
                        this.searchLocation()
                    }, ()=>{
                        console.log("error fetching the location data")
                    }
                )
            } else {
                console.log("error fetching location")
            }
        },
        searchLocation: function(){
            console.log(`searched address is ${this.address}`)
            this.geocoder.geocode({'address': this.address}, (results, status)=>{
                if (status == 'OK'){
                    console.log(`searched result si ${results}`)
                    console.log(results[0].geometry.location.lat(), results[0].geometry.location.lng())
                    this.fetchTime(results[0].geometry.location.lat(), results[0].geometry.location.lng())
                    this.setMarker(results)
                } else {
                     alert('Geocode was not successful for the following reason: ' + status);
                }
            })
        }, 
        setMarker:function(results){
                // 
                let historyItem = results[0]
                console.log(`result is ${results[0]}`)
                historyItem.selection=false //for the use of cancellation selection
                this.searchedHistory.push(historyItem)


            //  this.map.setCenter(results[0].geometry.location);
             if (this.marker){
                console.log("clear up function ")
                this.marker.setMap(null)
                this.marker = null
                console.log("cleared marker:", this.marker)
             }
         
              this.marker = new google.maps.Marker({

                        position: results[0].geometry.location,
                        map: this.map
                        })
            console.log("marker: ", this.marker)  
        },
        fetchTime(lat, lng){
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
                    console.log("data is ", data)
                })
                .catch(error=>{console.log("error" + error)})
        },
        deleteSelectedLocations:function(){
            this.searchedHistory.length > 0 
                ? this.searchedHistory = this.searchedHistory.filter((item)=>item.selection === false) 
                : alert("searched history list is empty, there is nothing to delete")
        }
    }}


</script>

<style>
    #map{
    width:100%;
    height:500px;
    background-color: grey;
    }
</style>