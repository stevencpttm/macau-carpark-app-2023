<template>
  <f7-page ptr @ptr:refresh="refresh">
    <f7-navbar>
      <f7-nav-title>Macau Carpark Info</f7-nav-title>
      <f7-nav-right>
        <f7-link @click="locateUser">定位</f7-link>
      </f7-nav-right>
    </f7-navbar>
    <f7-block class="text-align-center" v-if="carparkData.length == 0">
      <f7-preloader :size="28"></f7-preloader>
    </f7-block>
    <f7-list v-else>
      <f7-list-item
        class="item-link"
        @click="f7router.navigate('/info/', { props: { carpark: carpark } })"
        :key="index"
        :title="carpark.name"
        :footer="`🚙 ${carpark.car} 🛵 ${carpark.motor}`"
        v-for="(carpark, index) in carparkData"
      >
        <f7-icon icon="demo-list-icon"></f7-icon>
      </f7-list-item>
    </f7-list>
  </f7-page>
</template>

<script>
import { f7ready } from "framework7-vue";

export default {
  props: {
    f7router: Object,
  },
  data() {
    return {
      carparkData: [],
      location: {
        lat: null,
        lng: null,
      },
    };
  },
  watch: {
    locationCombined() {
      // eslint-disable-next-line no-console
      console.log(`location is being updated`);

      this.fetchData()
        .then((data) => {
          this.carparkData = data;
        })
        .catch((err) => {
          // eslint-disable-next-line no-console
          console.log(err);
        });
    },
  },
  computed: {
    locationCombined() {
      return this.location.lat && this.location.lng;
    },
  },
  mounted() {
    f7ready(async () => {
      this.fetchData().then((data) => {
        this.carparkData = data;
      });
    });
  },
  methods: {
    locateUser() {
      navigator.geolocation.getCurrentPosition((location) => {
        // eslint-disable-next-line no-console
        console.log(location);
        this.location = {
          lat: location.coords.latitude,
          lng: location.coords.longitude,
        };
      });
    },
    refresh(done) {
      console.log("refresh triggered");

      this.fetchData().then((data) => {
        this.carparkData = data;
        done();
      });
    },
    fetchData() {
      return new Promise((resolve, reject) => {
        let url = `http://localhost:3000`;

        if (this.location.lat != null && this.location.lng != null) {
          url += `?lat=${this.location.lat}&lng=${this.location.lng}`;
        }

        fetch(url)
          .then((res) => {
            res.json().then((json) => {
              resolve(json);
            });
          })
          .catch((err) => {
            reject(err);
          });
      });
    },
  },
};
</script>
