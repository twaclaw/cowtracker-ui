<template>
  <b-container fluid="xl">
    <b-row align-v="stretch">
      <b-col>
        <b-dropdown text="Select" class="m-md-2">
          <b-dropdown-item @click="get_selected_cow('all')"
            >all</b-dropdown-item
          >
          <b-dropdown-item
            v-for="(name, index) in cow_names"
            :key="index"
            @click="get_selected_cow(name)"
          >
            {{ name }}
          </b-dropdown-item>
        </b-dropdown>
      </b-col>
      <b-col>
        <b-form-checkbox
          class="m-md-2"
          id="checkbox-1"
          v-model="display_landmarks"
          name="checkbox-1"
        >
          Landmarks
        </b-form-checkbox>
      </b-col>
    </b-row>

    <div id="map-wrap" style="height: 90vh">
      <no-ssr>
        <cow-info></cow-info>
        <l-map :zoom="16" :maxZoom="17" :center="map_center">
          <l-tile-layer :url="layer_tile"></l-tile-layer>

          <!-- landmarks -->
          <l-marker
            :visible="display_landmarks"
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

          <!-- Cow history  -->
          <l-marker
            v-for="(cow, index) in cow_history"
            :key="'cow_his' + index"
            :lat-lng="[cow.pos.lat, cow.pos.lon]"
            :icon="get_icon('map-marker.png')"
          >
            <l-popup
              :content="when_seen(index, cow)"
              :options="{ autoClose: false, closeOnClick: false }"
            ></l-popup>
          </l-marker>

          <!-- Cows -->
          <l-marker
            v-for="(cow, index) in all_cows"
            :key="'cow' + index"
            :lat-lng="[cow.pos.lat, cow.pos.lon]"
            :icon="get_icon('cow-' + cow.status + '.png')"
            @click="marker_click(cow)"
          >
          </l-marker>

          <cow-info
            :meas="selected_cow"
            :visible="visible_cow_modal !== null"
            @hide="hide_modal()"
          ></cow-info>
        </l-map>
      </no-ssr>
    </div>
  </b-container>
</template>

<script>
import landmarks from "../static/landmarks.json";

export default {
  data() {
    return {
      map_center: [6.734878, -72.7725842],
      visible_cow_modal: null,
      selected_cow: null,
      layer_tile:
        "http://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}",
      marks: [],
      cow_names: [],
      all_cows: [],
      cow_history: [],
      display_landmarks: false,
    };
  },

  methods: {
    async fetch_names() {
      await this.$axios.$get("/api/v1/names").then((response) => {
        this.cow_names = response;
      });
    },
    async fetch_all_cow_coords() {
      await this.$axios.$get("/api/v1/meas/all").then((data) => {
        this.all_cows = data;
        this.cow_history = [];
      });
    },
    async fetch_cow_coords_history(name) {
      await this.$axios.$get("/api/v1/meas/" + name).then((data) => {
        this.all_cows = [data[0]];
        this.cow_history = data.slice(1);
      });
    },
    when_seen: function (index, cow) {
      var tz = this.$moment.tz.guess();
      var cowutc = this.$moment.unix(cow.t).utc();
      var from_now = cowutc.fromNow();

      return "[-" + (index + 1) + "] " + from_now;
    },
    get_icon: function (name) {
      let icon_name = "marker.png";
      if (name !== null && name !== undefined) {
        icon_name = name;
      }
      let x = 18;
      let y = 18;
      return new this.$L.Icon({
        iconUrl: "icons/" + icon_name,
        iconSize: [x, y],
        iconAnchor: [x - x / 2, y - y / 2],
      });
    },
    load_landmarks: function (json) {
      for (var p = 0; p < json.length; p++) {
        var point = json[p];
        this.marks.push({
          point: [point["lat"], point["lon"]],
          label: point["name"],
          icon: point["icon"],
        });
      }
    },
    display_modal: function (cow) {
      this.selected_cow = cow;
      this.visible_cow_modal = cow.name;
    },
    hide_modal: function () {
      this.visible_cow_modal = null;
    },
    marker_click: function (cow) {
      this.display_modal(cow);
    },
    get_selected_cow: function (name) {
      if (name === "all") {
        this.fetch_all_cow_coords();
      } else {
        this.fetch_cow_coords_history(name);
      }
    },
  },
  mounted() {
    this.load_landmarks(landmarks);
    this.fetch_names();
    this.get_selected_cow("all");
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
