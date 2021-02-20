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

    <b-alert
      v-for="(w, index) in warnings"
      :key="index"
      :variant="w.variant"
      show
      dismissible
      >{{ w.msg }}</b-alert
    >

    <div class="d-block text-center">
      <b-avatar
        badge
        :badge-variant="status"
        size="10em"
        variant="primary"
        :src="avatar"
      ></b-avatar>
    </div>

    <b-list-group-item>
      <p class="h5 mb-1">
        <b-icon icon="clock" :variant="time_variant"></b-icon>
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
          <b-icon :icon="battery_icon" :variant="battery_variant"></b-icon>
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
import "moment/locale/es";
export default {
  data() {
    return {
      battery_icon: "battery-full",
      battery_variant: "",
      time_variant: "",
    };
  },
  props: {
    meas: Object,
    visible: Boolean,
  },
  methods: {
    set_lang: function (lang) {
      this.$moment.locale(lang);
    },
  },
  mounted() {
    this.set_lang("es");
  },
  computed: {
    name() {
      if (this.meas) {
        return this.meas.name;
      }
    },
    avatar() {
      if (this.meas) {
        return "images/" + this.meas.name + ".jpg";
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
        var format = "HH:mm MMMM DD";
        var timestamp = this.$moment.unix(this.meas.t);
        var utcOffset = this.$moment().utcOffset();
        var local_time = timestamp.add(utcOffset, "minutes");
        var dateString = local_time.format(format);
        var from_now = local_time.fromNow();
        return from_now + " (" + dateString + ")";
      }
      return "";
    },
    radio_quality() {
      return this.meas
        ? "snr: " + this.meas.snr + ", rssi: " + this.meas.rssi
        : "";
    },
    status() {
      return this.meas ? this.meas.status : "info";
    },
    warnings() {
      if (this.meas && this.meas.warnings) {
        let warns = [];
        for (let wi = 0; wi < this.meas.warnings.length; wi++) {
          let w = this.meas.warnings[wi];
          if (w.code === "WARN_NO_MSGS_RECV") {
            this.time_variant = w.variant;
          } else if (w.code === "WARN_BATT_LOW") {
            this.battery_variant = w.variant;
            if (w.variant === "warning") {
              this.battery_icon = "battery-half";
            } else if (w.variant === "danger") {
              this.battery_icon = "battery-half";
            }
          } else if (w.code === "WARN_COW_NOT_MOVING") {
            warns.push({
              variant: w.variant,
              msg: "El animal no se mueve hace más de" + w.value + "h",
            });
          } else if (w.code === "WARN_COW_TOO_FAR") {
            warns.push({
              variant: w.variant,
              msg: "El animal está a más de " + w.value + "m de la antena.",
            });
          }
        }
        return warns;
      }
    },
  },
};
</script> 

<style scoped>
.b-icon {
  margin-right: 4px;
}
</style>