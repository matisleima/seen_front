<template>
  <div class="row m-3">
<div class="col">
</div>
<div class="col">
  <div id="map"></div>
</div>
<div class="col">
</div>
  </div>

</template>

<script>
import L from 'leaflet';

export default {
  name: 'YourComponentName',
  data() {
    return {
      map: null,
    };
  },
  mounted() {
    this.initMap();
  },
  methods: {
    initMap() {
      this.map = L.map('map').setView([58.62756196448109, 25.0117368536821], 7);
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '© OpenStreetMap contributors'
      }).addTo(this.map);
    }
  }
}

</script>

<style scoped>
#map {
  height: 400px;
  width: 600px;
}
</style>
