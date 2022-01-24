<template>
  <div class="flex flex-col h-screen max-h-screen">
    <!-- search/result -->
    <div class=" z-20 flex justify-center relative bg-hero-pattern bg-no-repeat bg-cover bg-center px-4 pt-8 pb-32">
      <!-- search input -->
      <div class="w-full max-w-screen-sm">
        <h1 class="text-white text-center text-3xl pb-4">IP Address Tracker</h1>
        <div class="flex">
          <input v-model="queryIp"  
            type="text" 
            class="flex-1 py-3 px-2 rounded-tl-md rounded-bl-md focus:outline-none"
            placeholder="Search for any IP address or leave empty to get your IP info">
            <i @click="getIpInfo" class="cursor-pointer bg-black text-white 
            px-4 rounded-tr-md rounded-br-md flex items-center
            fas fa-chevron-right"></i>
        </div>
      </div>
      <!-- IP info -->
      <IPInfo v-if="ipInfo" v-bind:ipInfo="ipInfo" />
    </div>
    <!-- map -->
    <div id="mapid" class="h-full z-10"></div>
  </div>
</template>

<script>
import IPInfo from "../components/IPInfo.vue"
import leaflet from "leaflet"
import {onMounted, ref} from "vue"
import axios from "axios"
export default {
  name: 'Home',
  components: {
    IPInfo
  },
  setup() {
    let mymap;

    const queryIp = ref("")
    const ipInfo = ref(null)

    onMounted(()=>{
      mymap = leaflet.map('mapid').setView([42.5145, -83.0147], 9);

      leaflet.tileLayer('https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=pk.eyJ1IjoibW9tbzI2IiwiYSI6ImNreXJlYWxtMTBtMzgybnRnMmwwZTQwemcifQ.STlGLE-LrrZ7_ns480Ps0w', {
      attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
      maxZoom: 18,
      id: 'mapbox/streets-v11',
      tileSize: 512,
      zoomOffset: -1,
      accessToken: 'pk.eyJ1IjoibW9tbzI2IiwiYSI6ImNreXJlYWxtMTBtMzgybnRnMmwwZTQwemcifQ.STlGLE-LrrZ7_ns480Ps0w'
      }).addTo(mymap);
    });
    const getIpInfo = async() =>{
      try{
        const data = await axios.get(
          `https://geo.ipify.org/api/v2/country,city,vpn?apiKey=at_FNdVBeOZCHFR3FuVANyDfdA88GA5W&ipAddress=${queryIp.value}`
        );
        const result = data.data;
        ipInfo.value = {
          address: result.ip,
          state: result.location.region,
          timezone: result.location.timezone,
          isp: result.isp,
          lat: result.location.lat,
          lng: result.location.lng,
        };
        leaflet.marker([ipInfo.value.lat, ipInfo.value.lng]).addTo(mymap);
        mymap.setView([ipInfo.value.lat, ipInfo.value.lng], 13);
      } catch(err){
        alert(err.message)
      }
    }
    return{queryIp,ipInfo,getIpInfo}
  },
}
</script>
