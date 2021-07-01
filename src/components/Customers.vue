<template>
  <div>
    <v-alert
      elevation="3"
      type="success"
      :value="successAlert"
      transition="slide-y-transition"
    >{{ successMessage }}</v-alert>
    <v-alert
      elevation="3"
      type="error"
      :value="errorAlert"
      transition="slide-y-transition"
    >{{ errorMessage }}</v-alert>
    <v-data-table
      :headers="headers"
      :items="customers"
      class="elevation-1"
    >
    
    <template v-slot:top>
      <v-toolbar
        flat
      >
        <v-toolbar-title>Clientes</v-toolbar-title>
        <v-divider
          class="mx-4"
          inset
          vertical
        ></v-divider>
        <v-spacer></v-spacer>
        <v-dialog
          v-model="dialog"
          max-width="500px"
        >
          <template v-slot:activator="{ on, attrs }">
            <v-btn
              color="primary"
              dark
              class="mb-2"
              v-bind="attrs"
              v-on="on"
            >
              Agregar Cliente
            </v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="text-h5">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-col
                    cols="12"
                    sm="6"
                    md="4"
                  >
                    <v-text-field
                      v-model="editedItem.FullName"
                      label="Nombre"
                      autocomplete="off"
                    ></v-text-field>
                  </v-col>
                  <v-col
                    cols="12"
                    sm="6"
                    md="4"
                  >
                    <v-text-field
                      v-model="editedItem.Address"
                      label="Dirección"
                      autocomplete="off"
                    ></v-text-field>
                  </v-col>
                  <v-col
                    cols="12"
                    sm="6"
                    md="4"
                  >
                    <v-text-field
                      v-model="editedItem.Phone"
                      label="Teléfono"
                      autocomplete="off"
                    ></v-text-field>
                  </v-col>
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn
                color="blue darken-1"
                text
                @click="close"
              >
                Cancelar
              </v-btn>
              <v-btn
                color="blue darken-1"
                text
                @click="save"
              >
                Guardar
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-dialog v-model="modalContacts" max-width="800px">
              <Contacts :customerId="customerContacts" :key="customerContacts" />

        </v-dialog>
        <v-dialog v-model="dialogDelete" max-width="500px">
          <v-card>
            <v-card-title class="text-h6 justify-center mb-6">¿Está seguro de eliminar este cliente?</v-card-title>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="closeDelete">Cancelar</v-btn>
              <v-btn color="red" text @click="deleteItemConfirm">Confirmar</v-btn>
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    <template v-slot:item.actions="{ item }">
      <v-icon
        small
        class="mr-2"
        title="Editar cliente"
        @click="editItem(item)"
      >
        mdi-pencil
      </v-icon>
      <v-icon
        small
        title="Eliminar cliente"
        class="mr-2"
        @click="deleteItem(item)"
      >
        mdi-delete
      </v-icon>
      <v-icon
        small
        :title="Titulo"
        @click="seeContacts(item)"
      >
        mdi-eye
      </v-icon>
    </template>
    <template v-slot:no-data>
      <v-btn
        color="primary"
        @click="getCustomers"
      >
        Reset
      </v-btn>
    </template>
    </v-data-table>
  </div>
</template>

<script>
import axios from 'axios'
import Contacts from './Contacts.vue';

export default {
    data: () => ({
      Titulo: 'Ver contactos',
      dialog: false,
      dialogDelete: false,
      modalContacts: false,
      headers: [
        { text: 'Nombre', value: 'FullName' },
        { text: 'Dirección', value: 'Address' },
        { text: 'Telefono', value: 'Phone' },
        { text: 'Fecha creación', value: 'CreationDate' },
        { text: 'Actions', value: 'actions', sortable: false }
      ],
      customers: [],
      editedIndex: -1,
      editedItem: {
        FullName: '',
        Address: '',
        Phone: ''
      },
      defaultItem: {
        FullName: '',
        Address: '',
        Phone: ''
      },
      successAlert: false,
      successMessage: '',
      errorAlert: false,
      errorMessage: '',
      customerContacts: 0
    }),

    components: {
      Contacts
    },

    computed: {
      formTitle () {
        return this.editedIndex === -1 ? 'Nuevo cliente' : 'Editar cliente'
      },
    },

    watch: {
      dialog (val) {
        val || this.close()
      },
      dialogDelete (val) {
        val || this.closeDelete()
      },
      modalContacts (val) {
        val || this.closeModalContacts()
      },
      successAlert (val) {
        if (!val) return

        setTimeout(() => (this.successAlert = false), 2500)
      },
      errorAlert (val) {
        if (!val) return

        setTimeout(() => (this.errorAlert = false), 2500)
      },
    },

    created () {
      this.getCustomers()
    },

    methods: {
      getCustomers() {
        axios.get('https://localhost:44357/api/Customers/GetAllCustomers')
        .then((resp) => {
          this.customers = resp.data
        }) 
      },

      sendCustomer(customer) {
        axios.post('https://localhost:44357/api/Customers/InsertCustomer', customer)
        .then((resp) => {
            this.getCustomers()
            this.successMessage = resp.data
            this.successAlert = true
        }).catch((err) => {
          console.log(err.response)
          this.errorMessage = err.response.data
          this.errorAlert = true
        })
      },

      updateCustomer(customer) {
        axios.put('https://localhost:44357/api/Customers/UpdateCustomer', customer)
        .then((resp) => {
            this.getCustomers()
            this.successMessage = resp.data
            this.successAlert = true
        }).catch((err) => {
          console.log(err.response)
          this.errorMessage = err.response.data
          this.errorAlert = true
        })
      },

      deleteCustomer(customer) {
        axios.delete(`https://localhost:44357/api/Customers/DeleteCustomerById?Id_customer=${customer.Id_customer}`)
        .then((resp) => {
            this.getCustomers()
            this.successMessage = resp.data
            this.successAlert = true
        }).catch((err) => {
          console.log(err.response)
          this.errorMessage = err.response.data
          this.errorAlert = true
        })
      },

      editItem (item) {
        this.editedIndex = this.customers.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialog = true
      },

      deleteItem (item) {
        this.editedIndex = this.customers.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialogDelete = true
      },

      seeContacts (item) {
        this.editedIndex = this.customers.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.customerContacts = item.Id_customer
        this.modalContacts = true
      },

      deleteItemConfirm () {
        this.deleteCustomer(this.editedItem)
        this.closeDelete()
      },

      close () {
        this.dialog = false
        this.$nextTick(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        })
      },

      closeDelete () {
        this.dialogDelete = false
        this.$nextTick(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        })
      },

      closeModalContacts () {
        this.modalContacts = false
      },

      save () {
        if (this.editedIndex > -1) {
          this.updateCustomer(this.editedItem)
          
        } else {
          this.sendCustomer(this.editedItem)
        }
        this.close()
      },
    },
  }
</script>

<style>

</style>