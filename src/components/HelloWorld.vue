<template>
  <div class="container-fluid invoice_custom">
    <div class="row">
      <div class="col-6 mb-3">
        <b-button variant="success" class="mr-3" @click="$bvModal.show('modal-new-invoice')">Nueva factura</b-button>
        <b-modal class="modal-items" id="modal-new-invoice" size="xl" hide-footer>
          <template #modal-title>
            Crear nueva factura
          </template>
          <div class="d-block">
            <div class="row">
              <div class="col-12 col-md-6">
                <h5>Artículos</h5>
                <div class="row">
                  <div v-for="(item, index) in items" :key="index" class="col-md-4 mb-2">
                    <a href="#" class="link-custom" @click="selectItem(item.id, item.quantity, item.name, item.unit_value)">
                      <ItemCompnent :name="item.name" :value="item.unit_value" :quantity="item.quantity" :id="item.id"/>
                    </a>
                  </div>
                </div>
              </div>
              <div class="col-12 col-md-6">
                <h5>Factura</h5>
                <div v-for="(item, index) in form.items" :key="index">
                  <div class="row mb-1">
                    <div class="col-4 pt-2">
                      {{ item.name }}
                    </div>
                    <div class="col-3 pt-2">
                      {{ item.unit_value }}
                    </div>
                    <div class="col-4">
                      <b-form-spinbutton id="demo-sb" v-model="form.items[index].quantity" min="1" :max="item.total_quantity"></b-form-spinbutton>
                    </div>
                    <div class="col-1 pt-2">
                      {{ item.id }}
                    </div>
                  </div>
                </div>
                <div class="separator mt-2 mb-2"></div>
                <div>
                  <b-form @submit="onSubmit" @reset="onReset" v-if="show">
                    <b-form-group id="input-group-1" label="Emisor:" label-for="input-1">
                      <b-form-select
                        id="input-1"
                        v-model="form.receiver_id"
                        value-field="id"
                        text-field="name"
                        :options="receivers"
                        required
                      ></b-form-select>
                    </b-form-group>
                    <b-form-group id="input-group-2" label="Comprador:" label-for="input-2">
                      <b-form-select
                        id="input-2"
                        v-model="form.transmitter_id"
                        value-field="id"
                        text-field="name"
                        :options="transmitters"
                        required
                      ></b-form-select>
                    </b-form-group>
                    <b-form-group id="input-group-3" label="IVA:" label-for="input-3">
                      <b-form-select
                        id="input-3"
                        v-model="form.iva"
                        @change="calculateValue"
                        required
                      >
                        <b-form-select-option value="1">0%</b-form-select-option>
                        <b-form-select-option value="2">5%</b-form-select-option>
                        <b-form-select-option value="3">19%</b-form-select-option>
                      </b-form-select>
                    </b-form-group>
                    <div class="text-right">
                      <strong>Valor sin iva:</strong><span>{{ form.value }}</span> <br>
                      <strong>IVA:</strong><span>{{ iva_value }}</span> <br>
                      <strong>Valor Total:</strong><span>{{ form.total_value }}</span> <br>
                    </div>
                    <div class="text-center">
                      <b-button type="submit" variant="primary" class="mr-2">Facturar</b-button>
                      <b-button type="reset" variant="danger">Cerrar</b-button>
                    </div>
                  </b-form>
                </div>
              </div>
            </div>
          </div>
        </b-modal>
      </div>
      <div class="col-6 text-right mb-3">
        <b-button @click="logout()" variant="danger">Cerrar sesion</b-button>
      </div>
    </div>
    <b-card
      border-variant="info"
      :header="msg"
      header-bg-variant="info"
      header-text-variant="white"
      align="left"
    >
      <b-card-text>
        <!-- <b-form @submit="onSubmit" v-if="show">
          <b-row>
            <b-form-group
              id="input-group-1"
              label="Filtro"
              label-for="input-1"
              class="col-md-3"
            >
              <b-form-input
                id="input-1"
                v-model="form.name"
                type="text"
                placeholder="Ej. Producto"
                required
              ></b-form-input>
            </b-form-group>

            <b-col class="d-flex align-items-center">
              <div class="mt-md-3">
                <b-button type="submit" variant="info" class="mr-3"
                  >Buscar</b-button
                >
              </div>
            </b-col>
          </b-row>
        </b-form> -->
        <div class="mt-3">
          <b-table
            striped
            hover
            responsive
            :items="invoices"
            :fields="fields"
            :per-page="perPage"
            :current-page="currentPage"
          >
            <template v-slot:cell(actions)="row">
              <b-button
                class="btn-rounded btn-bordered-primary mr-1"
                size="sm"
                variant="primary"
                title="Ver"
                @click="showTypeForm(row.item, true)"
                ><i class="mdi mdi-eye-circle" /><span
                  >Ver</span
                ></b-button
              >
              <b-button
                class="btn-rounded btn-bordered-warning"
                size="sm"
                variant="warning"
                title="Editar"
                @click="showTypeForm(row.item, false)">
                  <i class="mdi mdi-square-edit-outline" /><span>Editar</span>
              </b-button>
            </template>
          </b-table>
        </div>
        <div v-if="items.length > 0">
          <b-pagination
            v-model="currentPage"
            :total-rows="rows"
            :per-page="perPage"
            aria-controls="my-table"
            align="right"
            variant="info"
          ></b-pagination>
        </div>
      </b-card-text>
    </b-card>
  </div>
</template>

<script>
import ItemCompnent from './Item.vue'
export default {
  name: "index",
  props: {
    msg: String,
  },
  components: {
    ItemCompnent
  },
  mounted() {
    if (localStorage.getItem("token")) {
      console.log(localStorage.getItem("token"));
      this.axios.get('invoices')
      .then(res => {
        this.invoices=res.data
      })
      .catch(err => {
        console.error(err); 
      })
      this.axios.get('items')
      .then(res => {
        this.items=res.data
      })
      .catch(err => {
        console.error(err); 
      })
      this.axios.get('transmitters')
      .then(res => {
        this.transmitters=res.data
      })
      .catch(err => {
        console.error(err); 
      })
      this.axios.get('receivers')
      .then(res => {
        this.receivers=res.data
      })
      .catch(err => {
        console.error(err); 
      })
    } else {
      this.$router.push("/login");
    }
  },
  data() {
    return {
      form: {
        value: 0,
        iva: null,
        total_value: 0,
        transmitter_id: null,
        receiver_id: null,
        items: []  
      },
      show: true,
      state: false,
      fields: [
        {
          key: "number",
          label: "Numero de factura",
          sortable: true,
        },
        {
          key: "created_at",
          label: "Fecha y hora",
          sortable: false,
        },
        {
          key: "transmitter",
          label: "Emisor",
          sortable: true,
          formatter: (value) => {
            return value.name
          },
        },
        {
          key: "receiver",
          label: "Receptor",
          sortable: true,
          formatter: (value) => {
            return value.name
          },
        },
        {
          key: "value",
          label: "Valor neto",
          sortable: true,
          formatter: (value) => {
            return '$ '+value
          },
        },
        {
          key: "iva",
          label: "IVA",
          sortable: true,
        },
        {
          key: "total_value",
          label: "Valor a pagar",
          sortable: true,
          formatter: (value) => {
            return '$ '+value
          },
        },
        {
          key: "actions",
          label: "Acciones",
        },
      ],
      perPage: 10,
      currentPage: 1,
      items: [],
      invoices: [],
      receivers: [],
      transmitters: [],
      partial_value: 0,
      iva_value: 0,
      total_value: 0
    };
  },
  methods: {
    onSubmit(event) {
      event.preventDefault();
      let me = this;
      if (me.form.items.length > 0) {      
        this.axios
          .post("invoices", me.form)
          // eslint-disable-next-line no-unused-vars
          .then((response) => {      
            this.axios.get('invoices')
              .then(res => {
                this.invoices=res.data
              })
              .catch(err => {
                console.error(err); 
              })
            this.$bvModal.hide('modal-new-invoice')
            this.$swal({
              toast: true,
              position: "top-end",
              icon: "success",
              title: "Exitoso con resultados",
              showConfirmButton: false,
              timer: 2000,
            });
          });
      } else {
        this.$swal({
            toast: true,
            position: "top-end",
            icon: "error",
            title: "Falta seleccionar algun artículo",
            showConfirmButton: false,
            timer: 2000,
          });
      }
    },
    logout() {
      localStorage.removeItem("token");
      this.$router.push("/login");
    },
    selectItem(id, quantity, name, unit_value) {
      console.log(id + "-" + quantity)
      let find=this.form.items.findIndex(el=>el.id==id)
      if (find > -1) {
        let q = this.form.items[find]['quantity']
        let nq = q + 1;
        this.form.items[find]['quantity'] = nq
        this.partial_value =0
      } else {
        let newItem = {
          'id': id,
          'quantity': 1,
          'total_quantity': quantity,
          'name': name,
          'unit_value': unit_value
        }
        this.form.items.push(newItem)
        console.log(this.form.items)
      }
      let i = 0
      if (this.form.iva == 2) {
        i = 0.05
      } else if (this.form.iva == 3) {
        i = 0.19
      }
      let vtsi = 0
      this.form.items.forEach(element => {
        let v = element.unit_value * element.quantity
        vtsi+=v
      });
      this.form.value = vtsi
      this.iva_value = vtsi*i
      this.form.total_value = vtsi*(i+1)
    },
    calculateValue() {
      let i = 0
      if (this.form.iva == 2) {
        i = 0.04
      } else if (this.form.iva == 3) {
        i = 0.19
      }
      let vtsi = 0
      this.form.items.forEach(element => {
        let v = element.unit_value * element.quantity
        vtsi+=v
      });
      this.form.value = vtsi
      this.iva_value = vtsi*i
      this.form.total_value = vtsi*(i+1)
    },
    onReset(event) {
      event.preventDefault()
      this.$bvModal.hide('modal-new-invoice')
      // Reset our form values
      this.form.value = 0
      this.form.iva = null
      this.form.total_value = 0
      this.form.transmitter_id = null
      this.form.receiver_id = null
      this.form.items = []      
    },
    showTypeForm(row, type) {
      alert((type == 1 ? " Ver " : " Editar ") + row.id + " - " + row.number)
    }
  },
  computed: {
    rows() {
      return this.items.length;
    },
  },
};
</script>
<style lang="css">  
  .link-custom {    
    text-decoration: none !important;
    color: black !important;
  }
  .link-custom :hover .card {
    box-shadow: 0px 0px 10px #bbbbbb;
  }
  .separator {
    height: 2px;
    width: 100%;
    background: rgb(194, 193, 193);
  }
</style>
