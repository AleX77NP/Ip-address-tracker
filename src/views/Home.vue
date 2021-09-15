<template>
  <div class="flex flex-col h-screen max-h-screen">
    <div class="z-20 flex justify-center relative bg-hero-pattern bg-cover px-4 pt-8 pb-32">
      <div class="w-full max-w-screen-sm">
        <h1 class="text-white text-center text-3xl pb-4">IP Address Tracker</h1>
        <div class="flex">
          <input type="text" 
          v-model="queryIp"
          placeholder="Search for any IP Address or leave empty to get yout ip info"
          class="flex-1 py-3 px-2 rounded-tl-md rounded-bl-md focus:outline-none" />
          <i 
          @click="getIpInfo"
          class="cursor-pointer bg-black text-white px-4 rounded-tr-md 
          flex
          items-center
          bg-red-600
          rounded-br-md fas fa-chevron-right"></i>
        </div>
      </div>
      <IPInfo v-if="ipInfo" v-bind:ipInfo="ipInfo" />
    </div>

    <div id="mapid" class="h-full z-10">

    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import IPInfo from '../components/IPInfo.vue'
import leaflet from "leaflet"
import axios from 'axios'
import { onMounted, ref } from '@vue/runtime-core';
export default {
  name: 'Home',
  components: {
    IPInfo
  },
  setup() {
    let mymap;
    const queryIp = ref("")
    const ipInfo = ref(null)
    onMounted(() => {
      mymap = leaflet.map('mapid').setView([43.1407, 20.5181], 13);

      leaflet.tileLayer('https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=token', {
          attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
          maxZoom: 18,
          id: 'mapbox/streets-v11',
          tileSize: 512,
          zoomOffset: -1,
          accessToken: 'token'
      }).addTo(mymap);
    })

    const getIpInfo = async() => {
      try {
        const response = await axios.get(`https://geo.ipify.org/api/v1?apiKey=key&ipAddress=${queryIp.value}`)
        const result = response.data
        ipInfo.value = {
          address: result.ip,
          state: result.location.region,
          timezone: result.location.timezone,
          isp: result.isp,
          lat: result.location.lat,
          lng: result.location.lng
        }
        leaflet.marker([result.location.lat, result.location.lng]).addTo(mymap);
        mymap.setView([result.location.lat, result.location.lng],13);
      } catch(err) {
        console.log(err)
        alert(err.message)
      }
    }
    return { queryIp, ipInfo, getIpInfo }
  },
}
</script>
