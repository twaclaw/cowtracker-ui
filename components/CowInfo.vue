<template>
  <b-modal
    :visible="visible"
    @hide="$emit('hide', $event)"
    centered
    hide-footer
  >
    <template #modal-title
      ><h3 class="d-block text-center">{{ name }}</h3>
    </template>
    <div class="d-block text-center">
      <b-avatar
        badge
        badge-variant="warning"
        size="10em"
        variant="primary"
        :src="avatar"
      ></b-avatar>
    </div>

    <b-list-group-item>
      <p class="h5 mb-1">
        <b-icon icon="clock"></b-icon>
        {{ last_seen }}
      </p>
    </b-list-group-item>

    <b-list-group>
      <b-list-group-item>
        <p class="h5 mb-1">
          <b-icon icon="geo-fill"></b-icon>
          {{ position }}
        </p>
      </b-list-group-item>

      <b-list-group-item>
        <p class="h5 mb-1">
          <b-icon icon="thermometer"></b-icon>
          {{ temperature }}
        </p>
      </b-list-group-item>
      <b-list-group-item>
        <p class="h5 mb-1">
          <b-icon icon="battery-full" :variant="battery_variant"></b-icon>
          {{ battery }}
        </p>
      </b-list-group-item>
      <b-list-group-item>
        <p class="h5 mb-1">
          <b-icon icon="wifi"></b-icon>
          {{ radio_quality }}
        </p>
      </b-list-group-item>
    </b-list-group>
    <!-- <b-button class="mt-3" block @click="$bvModal.hide('cowinfo')"
      >Close</b-button
    > -->
  </b-modal>
</template>

<script>
export default {
  data() {
    return {
      battery_variant: "",
    };
  },
  props: {
    meas: Object,
    visible: Boolean,
  },
  computed: {
    name() {
      if (this.meas) {
        return this.meas.name;
      }
    },
    avatar() {
      if (this.meas) {
        return "images/"+this.meas.name+".jpg";
      }
    },
    battery() {
      if (this.meas) {
        return this.meas.batt_v + "V (" + this.meas.batt_cap + " %)";
      }
      return "";
    },
    position() {
      return this.meas
        ? "(" + this.meas.pos.lat + "," + this.meas.pos.lon + ")"
        : "()";
    },
    temperature() {
      return this.meas ? this.meas.temp + " C" : "";
    },
    last_seen() {
      if (this.meas) {
        var utcSeconds = this.meas.t;
        var d = new Date(0);
        d.setUTCSeconds(utcSeconds);
        var options = {
          weekday: "long",
          year: "numeric",
          month: "long",
          day: "numeric",
        };
        return d.toLocaleString(options);
      }
      return "";
    },
    radio_quality() {
      return this.meas
        ? "snr: " + this.meas.snr + ", rssi: " + this.meas.rssi
        : "";
    },
  },
};
</script> 

<style scoped>
.b-icon {
  margin-right: 4px;
}
</style>