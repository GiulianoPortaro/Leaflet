<template>
  <l-map id="map" ref="map" />
</template>

<script>
    import * as L from "leaflet";
    import { AntPath } from 'leaflet-ant-path';

    export default {
      name: "leaflet-wrapper",
      props: {
        coords: {
          name: 'coords',
          type: Object,
          required: true,
          description: 'Default coordinates'
        },
        configuration: {
          name: 'configuration',
          type: Object,
          required: true,
          description: 'Default params of leaflet (zoom, maxZoom, minZoom)'
        },
        markers: {
          name: 'markers',
          type: Object,
          default: null,
          description: 'Set of colors and related markers to show on the map'
        },
        type: {
          name: 'type',
          type: String,
          default: 'markers',
          description: 'Type of map to be built'
        }
      },
      data() {
        return {
          markerGroup: null
        }
      },
      watch: {
        markers: {
          handler: function() {
            this.$refs.map.mapObject.removeEventListener('moveend');
            if(this.type === 'markers') this.buildMarkers();
            else if(this.type === 'roads') this.buildRoads();
          }
        },
        type: {
          handler: function(val) {
            this.$refs.map.mapObject.removeEventListener('moveend');
            if(val === 'markers') this.buildMarkers();
            else if(val === 'roads') this.buildRoads();
          }
        }
      },
      mounted() {
        this.$refs.map.mapObject.setView([this.coords.lat, this.coords.lon], this.configuration.zoom);

        L.tileLayer(
          'http://{s}.tile.osm.org/{z}/{x}/{y}.png', {
            maxZoom: this.configuration.maxZoom,
            minZoom: this.configuration.minZoom
          }
        ).addTo(this.$refs.map.mapObject);

        /* Use to fix library bug: allows to load all tiles correctly */
        setTimeout(this.onResize, 300);
        /* end */

        this.markerGroup = L.layerGroup().addTo(this.$refs.map.mapObject);

        if(this.type === 'markers') this.buildMarkers();
        else if(this.type === 'roads') this.buildRoads();
        else this.$destroy();
      },
      methods: {
        onResize() {
          this.$refs.map.mapObject.invalidateSize();
        },
        buildMap() {
          this.$refs.map.mapObject.off();
          this.$refs.map.mapObject.remove();
          this.$refs.map.mapObject = L.map('map');

          this.$refs.map.mapObject.setView([this.coords.lat, this.coords.lon], this.configuration.zoom);

          L.tileLayer(
            'http://{s}.tile.osm.org/{z}/{x}/{y}.png', {
              maxZoom: this.configuration.maxZoom,
              minZoom: this.configuration.minZoom
            }
          ).addTo(this.$refs.map.mapObject);

          /* Use to fix library bug: allows to load all tiles correctly */
          setTimeout(this.onResize, 300);
          /* end */

          this.markerGroup = L.layerGroup().addTo(this.$refs.map.mapObject);
        },
        defaultMarker() {

          this.noMarkers = L.marker(this.$refs.map.mapObject.getCenter(), {
            icon: L.divIcon({
              html: '<div style="background-color: #5f86ed; color: white; border: 2px solid #314479; text-align: center;' +
                'margin-left: -250px; height: 50px; margin-top: -25px; width: 500px; font-size: 16px">' +
                '<p style="padding-top: 12px;"><b>No GPS coordinates available</b></p></div>'
            })
          });

          this.$refs.map.mapObject.addEventListener("moveend", () => {
            this.noMarkers.setLatLng(this.$refs.map.mapObject.getCenter());
          }, false);

          return this.noMarkers;
        },
        buildMarkers() {

          this.buildMap();

          if(this.markers != null) {

            const setMarkers = [];

            let k = 0;

            for (let eui of Object.keys(this.markers.data)) {

              if(this.markers.data[eui].data.length >= 2) this.markers.data[eui].data.sort(function (a, b) {
                return Number(a.timestamp) - Number(b.timestamp);
              });

              const lat = this.markers.data[eui].data[this.markers.data[eui].data.length - 1].lat;
              const lon = this.markers.data[eui].data[this.markers.data[eui].data.length - 1].lon;
              const alt = this.markers.data[eui].data[this.markers.data[eui].data.length - 1].alt;
              const timestamp = new Date(this.markers.data[eui].data[this.markers.data[eui].data.length - 1].timestamp).toLocaleString();

              let data = "<b>EUI: </b>" + eui + "<br><b>Latitude: </b>" + lat + "<br><b>Longitude: </b>" +
                lon + "<br><b>Altitude: </b>" + alt + "<br><b>Timestamp: </b>" + timestamp;

              const marker = L.circleMarker([lat, lon], {
                color: this.markers.colors[k],
                radius: 32,
                fillOpacity: 0.5
              }).bindTooltip(data);

              this.markerGroup.addLayer(marker);
              setMarkers.push(marker);

              k++;
            }

            const group = L.featureGroup(setMarkers);
            this.$refs.map.mapObject.fitBounds(group.getBounds());

            if(Object.keys(this.markers.data).length === 1)
              this.$refs.map.mapObject.setZoom(this.configuration.zoom);
          }
          else this.markerGroup.addLayer(this.defaultMarker());
        },
        buildRoads: function () {

          this.buildMap();

          if (this.markers !== null) {

            let k = 0;

            const setMarkers = [];

            for (let eui of Object.keys(this.markers.data)) {

              if(this.markers.data[eui].data.length >= 2) this.markers.data[eui].data.sort(function (a, b) {
                return Number(a.timestamp) - Number(b.timestamp);
              });

              for (let i = 0; i < this.markers.data[eui].data.length; i++) {

                const lat = this.markers.data[eui].data[i].lat;
                const lon = this.markers.data[eui].data[i].lon;
                const alt = this.markers.data[eui].data[i].alt;
                const timestamp = new Date(this.markers.data[eui].data[i].timestamp).toLocaleString();

                if(i === 0) {
                  let data = "<b>Start Position</b><br><b>EUI: </b>" + eui + "<br><b>Latitude: </b>" + lat + "<br><b>Longitude: </b>" +
                    lon + "<br><b>Altitude: </b>" + alt + "<br><b>Timestamp: </b>" + timestamp;

                  const marker = L.circleMarker([lat, lon], {
                    color: this.markers.colors[k],
                    radius: 16,
                    fillOpacity: 0.5,
                    fillColor: 'blue'
                  }).bindTooltip(data);

                  this.markerGroup.addLayer(marker);
                  setMarkers.push(marker);
                }

                if(i < this.markers.data[eui].data.length - 1) {
                  const nextLat = this.markers.data[eui].data[i + 1].lat;
                  const nextLon = this.markers.data[eui].data[i + 1].lon;
                  const nextAlt = this.markers.data[eui].data[i + 1].alt;
                  const nextTimestamp = new Date(this.markers.data[eui].data[i + 1].timestamp).toLocaleString();

                  let nextData = "<b>EUI: </b>" + eui + "<br><b>Latitude: </b>" + nextLat + "<br><b>Longitude: </b>" +
                    nextLon + "<br><b>Altitude: </b>" + nextAlt + "<br><b>Timestamp: </b>" + nextTimestamp;

                  if(this.markers.data[eui].data.length - 2 === i) nextData = "<b>End Position</b><br>" + nextData;

                  const marker = L.circleMarker([nextLat, nextLon], {
                    color: this.markers.colors[k],
                    radius: 16,
                    fillOpacity: (this.markers.data[eui].data.length - 2 === i) ? 0.5 : 0.7,
                    fillColor: (this.markers.data[eui].data.length - 2 === i) ? 'red' : this.markers.colors[k]
                  }).bindTooltip(nextData);

                  this.markerGroup.addLayer(marker);
                  setMarkers.push(marker);

                  let pathData = "<b>EUI:</b> " + eui + "<br>" + "<b>FROM</b><br><b>Latitude:</b> " + lat + "<br><b>Longitude: </b>" + lon +
                    "<br><b>Altitude: </b>" + alt + "<br><b>Timestamp: </b>" + timestamp + "<br><b>TO</b><br><b>Latitude: </b>" + nextLat +
                    "<br><b>Longitude: </b>" + nextLon + "<br><b>Altitude: </b>" + nextAlt + "<br><b>Timestamp: </b>" + nextTimestamp;

                  const path = new AntPath([[lat, lon], [nextLat, nextLon]], {
                    delay: 400,
                    dashArray: [
                      10,
                      63
                    ],
                    weight: 10,
                    color: "#5d5d69",
                    pulseColor: this.markers.colors[k],
                    paused: false,
                    reverse: false,
                    hardwareAccelerated: true
                  }).bindTooltip(pathData, { sticky: true });

                  this.markerGroup.addLayer(path);
                  setMarkers.push(marker);
                }
              }

              k++;
            }

            const group = L.featureGroup(setMarkers);
            this.$refs.map.mapObject.fitBounds(group.getBounds());
          } else this.markerGroup.addLayer(this.defaultMarker());
        }
      }
    }
</script>

<style scoped>
  #map {
    height: inherit;
  }
</style>

