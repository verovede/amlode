<template>
  <section class="container-fluid">
    <NavBarBack />
    <div class="container">
      <h1 class="mt-5">Formulario de Modificación de DEA</h1>

      <div class="h4 bg-grey p-2 d-flex bd-highlight">
        <div class="flex-fill  ">ID: {{ mostrarDea.id }}</div>
        <div class="flex-fill text-right">
          DEA ACTIVO:
          <input
            style="width: 20px; height: 20px"
            type="checkbox"
            v-model="formData.active"
          />
          <button
            class="btn btn-danger text-center ml-4" @click="restarPuntos()"
            
          >
            Descontar Puntos
          </button>
        </div>
      </div>

      <vue-form :state="formState" @submit.prevent="enviar()">
        <div class="row">
           <div class="col-12">
            <!-- CAMPO address  -->
            <validate tag="div">
              <span style="font-weight: bold">Dirección</span>
              <input
                v-model.trim="formData.address"
                id="address"
                name="address"
                type="text"
                class="form-control mb-3"
                autocomplete="off"
                
              />

              <field-messages name="address" show="$dirty">
                <div class="alert alert-danger mt-1" slot="required">
                  Campo obligatorio
                </div>
              </field-messages>
            </validate>
            <!-- FIN CAMPO address  -->
          </div>
          <div class="col-4">
            <!-- CAMPO latitude  -->
            <validate tag="div">
              <span style="font-weight: bold">Latitud</span>
              <input
                v-model.trim="formData.latitude"
                id="latitude"
                name="latitude"
                type="text"
                class="form-control mb-3"
                autocomplete="off"
                required
                validatelatitude
              />

              <field-messages name="latitude" show="$dirty">
                <div class="alert alert-danger mt-1" slot="required">
                  Campo obligatorio
                </div>
                <div class="alert alert-danger mt-1" slot="validatelatitude">
                    Coordenada inválida
                  </div>
              </field-messages>
            </validate>
            <!-- FIN CAMPO latitude  -->
          </div>
          <div class="col-4">
            <!-- CAMPO longitude  -->
            <validate tag="div">
              <span style="font-weight: bold">Longitud</span>
              <input
                v-model.trim="formData.longitude"
                id="longitude"
                name="longitude"
                type="text"
                class="form-control mb-3"
                autocomplete="off"
                required
                validatelongitude
              />

              <field-messages name="longitude" show="$dirty">
                <div class="alert alert-danger mt-1" slot="required">
                  Campo obligatorio
                </div>
                <div class="alert alert-danger mt-1" slot="validatelongitude">
                    Coordenada inválida
                  </div>
              </field-messages>
            </validate>
            <!-- FIN CAMPO longitude  -->
          </div>
          <div class="col-4">
            <!-- CAMPO datestamp  -->
            <validate tag="div">
              <span style="font-weight: bold">Fecha de Registro</span>
              <input
                v-model.trim="formData.datestamp"
                id="datestamp"
                name="datestamp"
                type="text"
                class="form-control mb-3"
                autocomplete="off"
                readonly
              />

              <field-messages name="datestamp" show="$dirty">
                <div class="alert alert-danger mt-1" slot="required">
                  Campo obligatorio
                </div>
              </field-messages>
            </validate>
            <!-- FIN CAMPO datestamp  -->
          </div>
        </div>

        <!-- ENVIO -->
        <div class="text-center">
          <button
            class="btn btn-info my-3 text-center"
            :disabled="formState.$invalid"
          >
            Enviar
          </button>
        </div>
      </vue-form>

      <div
        class="modal"
        tabindex="-1"
        role="dialog"
        :style="{ display: mostrarDisplay() }"
      >
        <div class="modal-dialog" role="document">
          <div class="modal-content">
            <div class="modal-header">
              <h5 class="modal-title">ADVERTENCIA!</h5>
            </div>
            <div class="modal-body">
              <p>{{ this.msjModal }}</p>
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
import NavBarBack from "../componentesBackOffice/NavBarBack.vue";
import { mixinsBack } from "../mixinsBack";

export default {
  mixins: [mixinsBack],
  name: "src-componentes-formulario-editar-dea",
  props: ["id"],
  components: {
    NavBarBack,
  },

  mounted() {
    this.$store.dispatch("getDeaById", this.id);
  },

  updated() {},

  data() {
    return {
      formState: {},
      formData: this.getInicialData(),
      modalShow: false,
      msjModal: "",
    };
  },

  methods: {
    async enviar() {
      console.log({ ...this.formData });

      const dea = {
        id: this.id,        
        latitude: { type: "String", value: this.formData.latitude },
        longitude: { type: "String", value: this.formData.longitude },
        address: { type: "String", value: this.formData.address },
        active: { type: "Boolean", value: this.formData.active },
      };

      

      if (!(await this.datosValidos())) {
        console.log("ERROR DE REGISTRO!");
        this.modalShow = true;
        return;
      }

      let resu = await this.$store.dispatch("actualizarDea", dea);

      if (resu) {
        this.$router.push({
          path: "/deas",
        });
      } else {
        console.log("ERROR DE REGISTRO!");
      }
    },

    async restarPuntos() {
      const usuarios = await this.$store.dispatch("getUsuarios")
      const usuariosFiltrados = usuarios.filter(usuario => usuario.deas.value.filter(idDea => idDea == this.id).length >0)
      
      if(usuariosFiltrados.length >0){
        const idDeas = usuariosFiltrados[0].deas.value
        const newArray = idDeas.filter(idDea => idDea != this.id );
        
        const body = {
                points: { type: "Number", value: usuariosFiltrados[0].points.value - 50 },
                deas: { type: "StructuredValue", value: newArray }
        } 

      const resu = await this.$store.dispatch("actualizarUsuario", {id: usuariosFiltrados[0].id, body})

      if(resu){
        this.modalShow = true;
        this.msjModal = "Dea eliminado del usuario con éxito";
      }
    }else{
      this.modalShow = true;
      this.msjModal = "Dea ya eliminado en el usuario";
    }
    
    },

    getInicialData() {
      return {
        name: "",
        latitude: "",
        longitude: "",
        datestamp: "",
        active: "",
        address:"",
      };
    },

    async cargarForm(dea) {
      this.formData.id = dea.id;
      this.formData.address = dea.address.value;
      this.formData.latitude = dea.latitude.value;
      this.formData.longitude = dea.longitude.value;
      this.formData.datestamp = dea.datestamp.value;
      this.formData.active = dea.active.value;
    },

    mostrarDisplay() {
      let estilo = "none";
      if (this.modalShow) {
        estilo = "inline";
      }
      return estilo;
    },

    async datosValidos() {
      const dea = await this.$store.dispatch("getDeaById", this.id);
      if (dea == null) {
        this.msjModal = "ID no encontrado para ese DEA";
        return false;
      }
      return true;
    },
  },
  computed: {},
};
</script>

<style scoped lang="css">
</style>
