<template>
  <!-- 
============ THANKS TO JOHN KOMARNICKI =============

https://www.youtube.com/channel/UCr0D7PVNOHqB3Td7lVl_LKw

====================================================

 -->
  <div class="flex flex-col h-screen max-h-screen">
    <!-- search / results -->
    <div
      class="
        flex
        justify-center
        relative
        bg-hero-pattern bg-cover
        px-4
        pt-8
        pb-32
        z-20
      "
    >
      <!-- search input -->
      <div class="w-full max-w-screen-sm">
        <h1 class="text-white text-center text-3xl pb-4">IP TRACKER</h1>
        <div class="flex">
          <i
            @click="useCurrentIp"
            class="
              cursor-pointer
              bg-black
              text-white
              px-4
              mx-3
              rounded-md
              flex
              items-center
            "
            >current ip</i
          >
          <input
            v-model="queryIp"
            type="text"
            class="flex-1 py-3 px-2 rounded-md focus:outline-none"
            placeholder="search for any IP address to get the ip info"
          />
          <i
            @click="getIpInfo"
            class="
              cursor-pointer
              bg-black
              text-white
              px-4
              mx-3
              rounded-md
              flex
              items-center
              fas
              fa-chevron-right
            "
          ></i>
        </div>
      </div>
      <IPInfo v-if="ipInfo" :ipInfo="ipInfo" />
    </div>
    <!-- leaflet MAP -->
    <div id="map" class="h-full z-10"></div>
  </div>
</template>

<script>
// mapbox a üye olup token al
// leaflet quickstart guide da css için gerekli kodu public/index.html ye koy, sayfadaki div al,
// locasyondan ip tespiti için https://www.ipify.org/ üye olup token alacağız, bu API'ye axios ile ulaşacağız,
// @ is an alias to /src
import IPInfo from "../components/IPInfo.vue";
import leaflet from "leaflet";
import { onMounted, ref } from "@vue/runtime-core";
import axios from "axios";

export default {
  name: "Home",
  components: {
    IPInfo,
  },
  setup() {
    let mymap;

    const queryIp = ref("");
    const ipInfo = ref(null);
    onMounted(() => {
      // fetch("https://api.ipify.org/?format=json")
      //   .then((results) => results.json())
      //   .then((data) => {
      //     // console.log(data.ip);
      //     queryIp.value = data.ip;
      //     getIpInfo();
      //   });

      mymap = leaflet.map("map").setView([51.505, -0.09], 13);

      leaflet
        .tileLayer(
          "https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=pk.eyJ1Ijoia2ViZWxlIiwiYSI6ImNrdzdoYnIzajB6cnYyd3FsdXR4OTl4bGEifQ.RJVVJZrA_4l0tyFcaqrAGw",
          {
            attribution:
              'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
            maxZoom: 18,
            id: "mapbox/streets-v11",
            tileSize: 512,
            zoomOffset: -1,
            accessToken:
              "pk.eyJ1Ijoia2ViZWxlIiwiYSI6ImNrdzdoYnIzajB6cnYyd3FsdXR4OTl4bGEifQ.RJVVJZrA_4l0tyFcaqrAGw",
          }
        )
        .addTo(mymap);
    });

    const getIpInfo = async () => {
      try {
        const data = await axios.get(
          `https://geo.ipify.org/api/v2/country,city?apiKey=at_tUyoBqvjriTRIHMW5cqBcEU055HDr&ipAddress=${queryIp.value}`
        );
        const result = data.data;
        // console.log(result);
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
      } catch (err) {
        // alert(err.message);
        console.log(err);
      }
    };

    const useCurrentIp = () => {
      fetch("https://api.ipify.org/?format=json")
        .then((results) => results.json())
        .then((data) => {
          // console.log(data.ip);
          queryIp.value = data.ip;
          getIpInfo();
        });
    };

    return { queryIp, ipInfo, getIpInfo, useCurrentIp };
  },
};
</script>
