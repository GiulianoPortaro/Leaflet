<template>

  <v-container grid-list-lg style="max-width: 95%">

    <h1>Leaflet Wrapper Demo</h1>

    <v-layout row wrap class="content-info">

      <v-flex xs12 sm12 md7 lg7>
        <div class="map-container">
          <leaflet-wrapper
            :coords="defaultCoords"
            :configuration="configuration"
            :type="mapSelected"
            :markers="markers"
          ></leaflet-wrapper>
        </div>
      </v-flex>

      <v-flex xs12 sm12 md5 lg5>

        <h3>Props Type</h3>
        <p>
          The props type allows to change the type of map. The values allowed follow: "markers" to display the last
          position of one device and "roads" to display all the positions crossed by device
        </p>
        <v-btn :class="(mapSelected === 'roads') ? 'error' : 'info'" @click="mapSelected = 'roads'">Roads</v-btn>
        <v-btn :class="(mapSelected === 'markers') ? 'error' : 'info'" @click="mapSelected = 'markers'">Positions</v-btn>

        <hr>

        <h3>Randomize</h3>
        <p>Try the Leaflet Wrapper adding random data to one random device.</p>
        <v-btn @click="addData" class="success">Add random data</v-btn>

        <hr>

        <h3>Props Markers</h3>
        <p>
          The props coords it's an object with two main properties: colors and data. Colors is an array of string that
          describe the colors that each markers must have, for this reason it's length must has equals to the properties
          contains into 'data' property. The second property, data, is an object that have one property for each device
          and the property name is the device's EUI. Inside it has an array of data and each have: "lat", "lon" and
          "timestamp".
        </p>

        <v-textarea id="markers" rows="20" label="Markers" :value="JSON.stringify(markers, null, 2)"></v-textarea>
        <v-btn class="info" @click="change('markers')">Update Markers</v-btn>

        <hr>

        <h3>Props Coords</h3>
        <p>
          The props coords it's an object with two properties: lat and lon. This object is used to display the message
          "No GPS coordinates available" in the case that there aren't markers.
        </p>
        <strong>{{defaultCoords}}</strong>

        <hr>

        <h3>Props Configuration</h3>
        <p>
          The props coords it's an object with three properties: zoom, minZoom, maxZoom. This object it's used to
          configure the maximum and minimum zoom on the leaflet map and the default zoom.
        </p>
        <strong>{{configuration}}</strong>

      </v-flex>

    </v-layout>

  </v-container>

</template>

<script>
import LeafletWrapper from "@/components/LeafletWrapper";

export default {
  components: {LeafletWrapper},
  name: 'demo',
  data() {
    return {
      defaultCoords: {
        lat: 45.071838,
        lon: 7.657481
      },
      configuration: {
        zoom: 6,
        maxZoom: 18,
        minZoom: 2
      },
      mapTypes: [
        'markers',
        'roads'
      ],
      mapSelected: 'roads',
      markers: {
        colors: ['blue', 'green'],
        data: {
          "DEVICE1": {
            data: [
              {
                lat: 42,
                lon: 7,
                timestamp: 1557397902
              },
              {
                lat: 35,
                lon: 7,
                timestamp: 1557397502
              }
            ]
          },
          "DEVICE2": {
            data: [
              {
                lat: 43,
                lon: 4,
                timestamp: 1557394902
              }
            ]
          }
        }
      }
    }
  },
  methods: {
    change(name) {
      this[name] = JSON.parse(document.getElementById(name).value);
    },
    addData() {
      const keys = Object.keys(this.markers.data);
      const len = keys.length - 1;
      const i = Math.round(Math.random() * len);
      this.markers.data[keys[i]].data.push({
        lat: Math.random() * 50 + 3,
        lon: Math.random() * 70 + 2,
        timestamp: new Date().getTime() / 1000
      });
      this.markers = Object.assign({}, this.markers);

    }
  }
}
</script>

<style scoped>
  .map-container {
    height: 600px;
  }
  hr {
    margin-top: 10px;
    margin-bottom: 10px;
  }
</style>
