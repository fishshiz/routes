<template>
  <div>
    <div id="map"></div>
    <div class="controls">
      <input type="color" value="#ff0000" v-model="color" />
      <input type="range" min="1" max="40" value="9" class="slider" v-model="width" />
      <input type="range" min="0" max="1" value="0.4" step="0.1" class="slider" v-model="opacity" />
    </div>
  </div>
</template>

<script>
import mapboxgl from "mapbox-gl";
import polyline from "@mapbox/polyline";
import axios from "axios";
import randomColor from "randomcolor";
export default {
  name: "Map",
  data() {
    return {
      line: [],
      map: null,
      color: "#ff0000",
      width: 9,
      opacity: 0.4
    };
  },
  watch: {
    color() {
      this.redrawMap("color");
    },
    width() {
      this.redrawMap("width");
    },
    opacity() {
      this.redrawMap("opacity");
    }
  },
  mounted() {
    mapboxgl.accessToken =
      "pk.eyJ1IjoiZmlzaHNoaXoiLCJhIjoiY2plODhmbnpoMDdxczMzbGFhaTd2djV2aSJ9.EPRw6PUFY0o_lVTihWMCdA";
    this.map = new mapboxgl.Map({
      container: "map",
      zoom: 13,
      style: "mapbox://styles/fishshiz/cjeos1vrn5gt62ss8mh2px0id",
      center: [-121.28004, 37.87386],
      hash: true,
      visible: false,
      transformRequest: (url, resourceType) => {
        if (resourceType === "Source" && url.startsWith("http://myHost")) {
          return {
            url: url.replace("http", "https"),
            headers: { "my-custom-header": true },
            credentials: "include" // Include cookies for cross-origin requests
          };
        }
      }
    });

    axios
      .get("https://www.strava.com/api/v3/athlete/activities?per_page=100", {
        headers: {
          Authorization: "Bearer ed3060fac84dfcb8f58f76f4f2d260d9f39c014d"
        }
      })
      .then(res => {
        let pol;
        let line;
        // let colors = randomColor({
        //   luminosity: "light",
        //   hue: "orange",
        //   count: 3
        // });
        res.data.forEach((datum, id) => {
          pol = datum.map.summary_polyline;
          line = polyline.toGeoJSON(pol);
          this.line.push(line);

          this.map.addLayer({
            id: `route-${id}`,
            type: "line",
            source: {
              type: "geojson",
              data: {
                type: "Feature",
                properties: {},
                opactity: 0.5,
                geometry: line
              }
            },
            layout: {
              "line-join": "round",
              "line-cap": "round"
            },
            paint: {
              "line-color": `${this.color}`,
              "line-width": 13,
              "line-opacity": 0.4
            }
          });
        });
      });
  },
  methods: {
    redrawMap(type) {
      this.line.forEach((line, id) => {
        switch (type) {
          case "color":
            this.map.setPaintProperty(`route-${id}`, "line-color", this.color);
            break;
          case "width":
            this.map.setPaintProperty(
              `route-${id}`,
              "line-width",
              parseInt(this.width)
            );
            break;
          case "opacity":
            this.map.setPaintProperty(
              `route-${id}`,
              "line-opacity",
              parseFloat(this.opacity)
            );
            break;
        }
      });
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#map {
  position: absolute;
  top: 0px;
  bottom: 0px;
  width: 100%;
}

.controls {
  background-color: aqua;
  z-index: 23;
  height: 300px;
  width: 400px;
  position: absolute;
  top: 20px;
  display: flex;
  flex-direction: column;
  justify-content: space-around;
  align-items: center;
}
input {
  width: 80%;
  height: 20%;
}
</style>
