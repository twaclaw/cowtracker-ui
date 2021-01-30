<template>
  <div id="map-wrap" style="height: 100vh">
    <no-ssr>
      <l-map :zoom="16" :center="map_center">
        <l-tile-layer :url="layer_tile"></l-tile-layer>

        <l-marker
          v-for="(marker, index) in marks"
          :key="index"
          :lat-lng="marker.point"
          :icon="get_icon(marker.icon)"
        >
          <l-popup
            :content="marker.label"
            :options="{ autoClose: false, closeOnClick: false }"
          ></l-popup>
        </l-marker>
      </l-map>
    </no-ssr>
  </div>
</template>

<script>
import landmarks from "../static/landmarks.json";

export default {
  data() {
    return {
      map_center: [6.74, -72.78],
      layer_tile:
        "http://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}",
      marks: [],
    };
  },
  methods: {
    get_icon: function (name) {
      let icon_name = "marker.png";
      if (name !== null && name !== undefined){
        icon_name = name;
      }
      return new this.$L.Icon({
        iconUrl: "icons/"+icon_name,
        iconSize: [40, 40],
        iconAnchor: [21, 35],
      })
    },
    load_landmarks: function (json) {
      for (var p = 0; p < json.length; p++) {
        var point = json[p];
        this.marks.push({
          point: [point["lat"], point["lon"]],
          label: point["name"],
          icon: point["icon"]
        });
      }
    },
  },
  mounted() {
    this.load_landmarks(landmarks);
  },
};
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: "Quicksand", "Source Sans Pro", -apple-system, BlinkMacSystemFont,
    "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>
