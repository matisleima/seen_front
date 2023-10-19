<template>
  <div @click="handleBackgroundClick" class="white-background">
    <div class="row m-3">
      <div class="col">
      </div>

      <div class="col">
        <div id="map" @click.stop></div>
      </div>

      <div class="col">
      </div>
    </div>
    <div class="row">

      <div class="col">
        <p>1. Klõpsa seenekoha märkimiseks kaardil!</p>
        <p>2. Kirjuta siia, milliseid seeni sealt leida võib!</p>
        <textarea v-model="descriptionBox" rows="5" cols="30" placeholder="Vöödilised võluseened..."
                  @click.stop @mousedown="startDrag"
                  @mouseup="endDrag"></textarea>
      </div>

      <div class="col m-3">
        <div>
          <button :class="{ 'disabled-button': marker === null }" :disabled="marker === null" @click.stop="savePoint" class="btn btn-success">
            SALVESTA
          </button>
        </div>
        <div>
          <button :class="{ 'disabled-button': pointIsSelected === false }" :disabled="pointIsSelected === false"
                  @click.stop="editPoint" type="button" class="btn btn-warning">MUUDA
          </button>
        </div>
        <div>
          <button :class="{ 'disabled-button': pointIsSelected === false }" :disabled="pointIsSelected === false"
                  @click.stop="deletePoint" type="button" class="btn btn-danger">KUSTUTA
          </button>
        </div>
        <div v-if="message.length > 0" class="alert alert-warning m-3"> {{ message }}</div>
      </div>
    </div>
  </div>
</template>

<script>
import L from 'leaflet';
import '@/assets/css/my-style.css';

export default {
  name: 'HomeView',
  data() {
    return {
      map: null,
      message: '',
      descriptionBox: '',
      selectedPointId: 0,
      selectedPoint: null,
      originalCoordinates: [],
      draggedCoordinates: [],
      pointIsSelected: false,
      isDragging: false,
      marker: null,
      markers: [],
      singlePoint: {
        type: 'Feature',
        geometry: {
          type: 'Point',
          coordinates: []
        },
        properties: {
          id: 0,
          description: ''
        }
      },
      allPoints: {
        type: 'FeatureCollection',
        features: [
          {
            type: 'Feature',
            geometry: {
              type: 'Point',
              coordinates: []
            },
            properties: {
              id: 0,
              description: ''
            }
          }
        ]
      }
    };
  },
  methods: {
    initMap() {
      const estoniaBounds = L.latLngBounds(
          L.latLng(57.5102, 21.3822),
          L.latLng(59.7040, 28.3799)
      );
      this.map = L.map('map', {
        minZoom: 7,
        maxBounds: estoniaBounds,
        maxBoundsViscosity: 1.0
      }).setView([58.62756, 25.01173], 7);
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '© OpenStreetMap contributors'
      }).addTo(this.map);
      this.map.on('click', this.addOnePointOnClick);
    },
    addOnePointOnClick(e) {
      if (this.marker) {
        this.map.removeLayer(this.marker);
      }
      this.singlePoint.geometry.coordinates = [e.latlng.lat, e.latlng.lng];
      this.marker = L.marker(this.singlePoint.geometry.coordinates).addTo(this.map);
      this.markers.forEach(marker => {
        marker.dragging.disable();
      });

      if (this.selectedPoint) {
        this.selectedPoint.setLatLng({
          lat: this.selectedPoint.originalCoordinates[0],
          lng: this.selectedPoint.originalCoordinates[1]
        });
        this.selectedPoint = null;
      }
    },
    populateMap() {
      this.clearNonTileLayers();

      this.markers = [];
      this.allPoints.features.forEach(feature => {
        const marker = this.createMarker(feature);
        this.markers.push(marker);
        marker.addTo(this.map);
      });
    },
    createMarker(feature) {
      const marker = L.marker(feature.geometry.coordinates);
      marker.originalCoordinates = [...feature.geometry.coordinates];

      marker.on('click', e => {
        if (this.marker) {
          this.map.removeLayer(this.marker);
          this.marker = null;
        }

        const popup = L.popup()
            .setLatLng(e.latlng)
            .setContent(feature.properties.description)
            .openOn(this.map);

        if (this.selectedPoint && this.selectedPoint.dragging) {
          this.selectedPoint.dragging.disable();
        }

        this.selectedPoint = marker;
        this.selectedPoint.dragging.enable();

        this.pointIsSelected = true;
        this.selectedPointId = feature.properties.id;
        this.originalCoordinates = feature.geometry.coordinates;
        this.descriptionBox = feature.properties.description;
      });
      marker.on('dragend', (e) => {
        const newLatLng = marker.getLatLng();
        this.draggedCoordinates = [newLatLng.lat, newLatLng.lng];
      });
      return marker;
    },
    clearNonTileLayers() {
      this.map.eachLayer(layer => {
        if (!(layer instanceof L.TileLayer)) {
          this.map.removeLayer(layer);
        }
      });
    },
    savePoint() {
      if (this.descriptionBox.length > 0) {
        this.singlePoint.properties.description = this.descriptionBox;
        this.$http.post("http://localhost:8080/add", this.singlePoint
        ).then(response => {
          this.message = 'Asukoht salvestatud'
          setTimeout(() => {
            this.message = ''
          }, 5000);
          this.getAllPoints()
          this.descriptionBox = ''
        }).catch(error => {
          const errorResponseBody = error.response.data
        })
      } else {
        this.message = 'Palun kirjelda asukohta!'
        setTimeout(() => {
          this.message = ''
        }, 5000);
      }
    },
    editPoint() {
      if (this.descriptionBox.length > 0) {
        this.singlePoint.properties.description = this.descriptionBox;
        this.singlePoint.properties.id = this.selectedPointId

        if (this.draggedCoordinates.length > 0) {
          this.singlePoint.geometry.coordinates = this.draggedCoordinates;
        } else {
          this.singlePoint.geometry.coordinates = this.originalCoordinates;
        }

        this.$http.put("http://localhost:8080/edit", this.singlePoint
        ).then(response => {
          this.message = 'Asukoht muudetud'
          setTimeout(() => {
            this.message = ''
          }, 5000);
          this.descriptionBox = ''
          this.map.closePopup();
          this.selectedPoint.dragging.disable();
          this.draggedCoordinates = [];
          this.selectedPoint = null;
          this.getAllPoints()

        }).catch(error => {
          const errorResponseBody = error.response.data
        })
      } else {
        this.message = 'Palun kirjelda asukohta!'
        setTimeout(() => {
          this.message = ''
        }, 5000);
      }
    },
    deletePoint() {
      if (this.pointIsSelected) {
        this.$http.delete("http://localhost:8080/delete", {
              params: {
                id: this.selectedPointId
              }
            }
        ).then(response => {
          this.message = 'Asukoht kustutatud'
          setTimeout(() => {
            this.message = ''
          }, 5000);
          this.getAllPoints()
          this.descriptionBox = ''
          this.selectedPointId = 0
          this.pointIsSelected = false
        }).catch(error => {
          const errorResponseBody = error.response.data
        })
      } else {
        this.message = 'Palun vali mõni asukoht!'
        setTimeout(() => {
          this.message = ''
        }, 5000);
      }
    },
    getAllPoints() {
      this.$http.get("http://localhost:8080/get-all")
          .then(response => {
            this.allPoints = response.data
            this.populateMap()
          })
          .catch(error => {
            const errorResponseBody = error.response.data
          })
    },
    startDrag() {
      this.isDragging = true;
    },
    endDrag() {
      this.isDragging = false;
    },
    handleBackgroundClick() {
      if (this.isDragging) {
        this.isDragging = false;
        return;
      }
      this.pointIsSelected = false
      if (this.marker) {
        this.map.removeLayer(this.marker);
        this.marker = null
      }
      this.map.closePopup();
      this.descriptionBox = '';
      if (this.selectedPoint) {
        this.selectedPoint.setLatLng({
          lat: this.selectedPoint.originalCoordinates[0],
          lng: this.selectedPoint.originalCoordinates[1]
        });
        this.descriptionBox = ''
        this.markers.forEach(marker => {
          marker.dragging.disable();
        });
      }
    }
  },
  mounted() {
    this.initMap();
    this.getAllPoints()
  }
}
</script>