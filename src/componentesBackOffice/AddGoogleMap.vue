<template>
  <section>
    <div>
      <gmap-map :center="center" :zoom="15" class="map">
        <gmap-marker
          v-for="(gmp, index) in deasActivos"
          :key="index"
          :position="armarPosition(gmp)"
          @click="setDea(gmp)"
        ></gmap-marker>
      </gmap-map>

      <!-- MODAL -->
      <div
        v-show="modalShow"
        class="modal"
        tabindex="-1"
        role="dialog"
        style="display: inline"
      >
        <div class="modal-dialog" role="document">
          <div class="modal-content">
            <div class="modal-header">
              <h5 class="modal-title">Información DEA</h5>
            </div>
            <div class="modal-body">
              <p>Latitud: {{ mostrarDatos.latitud }}</p>
              <p>Longitud: {{ mostrarDatos.longitud }}</p>
              <p>Dirección: {{ mostrarDatos.direccion }}</p>
              <p>Vigente desde: {{ mostrarDatos.fecha }}</p>
            </div>
            <div class="modal-footer">
              <button
                type="button"
                class="btn btn-secondary"
                @click="modalShow = false"
              >
                Cerrar
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>
 
<script>
export default {
  name: "DrawGoogleMap",

  props: ["deasActivos"],

  components: {},

  data() {
    return {
      center: {
        lat: -34.45055543390788,
        lng: -58.54288444037823,
      },
      locations: [],
      currentLocation: null,
      modalShow: false,
      msjModal: "",
      dea: "",
      latitud: "",
      longitud: "",
      direccion: "",
    };
  },

  mounted() {
    this.setLocationLatLng();
  },

  methods: {
    setPlace(loc) {
      this.currentLocation = loc;
      console.log("current location", loc);
    },

    addLocationMarker() {
      if (this.currentLocation) {
        const marker = {
          lat: this.currentLocation.geometry.location.lat(),
          lng: this.currentLocation.geometry.location.lng(),
        };
        this.locationMarkers.push({ position: marker });
        this.locPlaces.push(this.currentLocation);
        this.center = marker;
        this.existingPlace = null;
      }
    },

    armarPosition(dea) {
      let location = {
        lat: Number(dea.latitude.value),
        lng: Number(dea.longitude.value),
        label: "United States",
      };

      //this.setDea(dea);

      return location;
    },

    setDea(dea) {
      this.latitud = dea.latitude.value;
      this.longitud = dea.longitude.value;
      this.direccion = dea.address.value;
      this.fecha = dea.datestamp.value;
      console.log(this.dea);
      this.modalShow = true;
    },

    setLocationLatLng: function () {
      navigator.geolocation.getCurrentPosition((geolocation) => {
        this.center = {
          lat: geolocation.coords.latitude,
          lng: geolocation.coords.longitude,
        };
      });
    },
  },

  computed: {
    mostrarDatos() {
      return {
        latitud: this.latitud,
        longitud: this.longitud,
        direccion: this.direccion,
        fecha: this.fecha,
      };
    },
  },
};
</script>
<style scoped lang="css">
.map {
  width: 100%;
  height: 600px;
}
</style>