# demo-leaflet

> A Wrapper Leaflet for framework Vue.js

The props accepted by Wrapper following:

<b>Props Type</b>
<br>
The props type allows to change the type of map. The values allowed follow: "markers" to display the last position of one device and "roads" to display all the positions crossed by device

<b>Props Markers</b>
<br>
The props coords it's an object with two main properties: colors and data. Colors is an array of string that describe the colors that each markers must have, for this reason it's length must has equals to the properties contains into 'data' property. The second property, data, is an object that have one property for each device and the property name is the device's EUI. Inside it has an array of data and each have: "lat", "lon" and "timestamp".

<b>Props Coords</b>
<br>
The props coords it's an object with two properties: lat and lon. This object is used to display the message "No GPS coordinates available" in the case that there aren't markers.

{ "lat": 45.071838, "lon": 7.657481 }

<b>Props Configuration</b>
<br>
The props coords it's an object with three properties: zoom, minZoom, maxZoom. This object it's used to configure the maximum and minimum zoom on the leaflet map and the default zoom.

{ "zoom": 6, "maxZoom": 18, "minZoom": 2 }

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report
```

For a detailed explanation on how things work, check out the [guide](http://vuejs-templates.github.io/webpack/) and [docs for vue-loader](http://vuejs.github.io/vue-loader).
