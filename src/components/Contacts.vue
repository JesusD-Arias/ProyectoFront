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
      :items="contacts"
      :items-per-page="5"
      class="elevation-1"
    >
    
    <template v-slot:top>
      <v-toolbar
        flat
      >
        <v-toolbar-title>Contactos</v-toolbar-title>
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
              color="success"
              dark
              class="mb-2"
              v-bind="attrs"
              v-on="on"
            >
              Agregar Contacto
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
                      v-model="editedItem.FullName_contact"
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
                      v-model="editedItem.Address_contact"
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
                      v-model="editedItem.Phone_contact"
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
        <v-dialog v-model="dialogDelete" max-width="500px">
          <v-card>
            <v-card-title class="text-h8 justify-center mb-6">¿Está seguro de eliminar este contacto?</v-card-title>
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
        title="Editar contacto"
        @click="editItem(item)"
      >
        mdi-pencil
      </v-icon>
      <v-icon
        small
        title="Eliminar contacto"
        class="mr-2"
        @click="deleteItem(item)"
      >
        mdi-delete
      </v-icon>
    </template>
    <template v-slot:no-data>
      <v-btn
        color="primary"
        @click="getContactsByIdCustomer"
      >
        Reset
      </v-btn>
    </template>
    </v-data-table>
  </div>
</template>

<script>
import axios from 'axios'
export default {

    props: {
        customerId: {
            type: Number,
            default: null
        }
    },

    data: () => ({
      Titulo: 'Ver contactos',
      dialog: false,
      dialogDelete: false,
      headers: [
        { text: 'Nombre', value: 'FullName_contact' },
        { text: 'Dirección', value: 'Address_contact' },
        { text: 'Telefono', value: 'Phone_contact' },
        { text: 'Actions', value: 'actions', sortable: false }
      ],
      contacts: [],
      editedIndex: -1,
      editedItem: {
        FullName_contact: '',
        Address_contact: '',
        Phone_contact: ''
      },
      defaultItem: {
        FullName_contact: '',
        Address_contact: '',
        Phone_contact: ''
      },
      successAlert: false,
      successMessage: '',
      errorAlert: false,
      errorMessage: '',
      IdCustomer: 0
    }),

    computed: {
      formTitle () {
        return this.editedIndex === -1 ? 'Nuevo contacto' : 'Editar contacto'
      },
    },

    watch: {
      dialog (val) {
        val || this.close()
      },
      dialogDelete (val) {
        val || this.closeDelete()
      },
      successAlert (val) {
        if (!val) return

        setTimeout(() => (this.successAlert = false), 2500)
      },
      errorAlert (val) {
        if (!val) return

        setTimeout(() => (this.errorAlert = false), 2500)
      },
      customerId: {
        handler(newVal) {
            this.IdCustomer = newVal
        },
        immediate: true
      }
    },

    created () {
        if(this.customerId){
            this.IdCustomer = this.customerId
        }
        this.getContactsByIdCustomer()
    },

    methods: {
      getContactsByIdCustomer() {
        axios.get(`https://localhost:44357/api/Contacts/GetContactsByIdCustomer?Id_customer=${this.IdCustomer}`)
        .then((resp) => {
          this.contacts = resp.data
        }) 
      },

      sendContact(Contact) {
        axios.post('https://localhost:44357/api/Contacts/InsertContact', Contact)
        .then((resp) => {
            this.getContactsByIdCustomer()
            this.successMessage = resp.data
            this.successAlert = true
        }).catch((err) => {
          console.log(err.response)
          this.errorMessage = err.response.data
          this.errorAlert = true
        })
      },

      updateContact(Contact) {
        axios.put('https://localhost:44357/api/Contacts/UpdateContact', Contact)
        .then((resp) => {
            this.getContactsByIdCustomer()
            this.successMessage = resp.data
            this.successAlert = true
        }).catch((err) => {
          console.log(err.response)
          this.errorMessage = err.response.data
          this.errorAlert = true
        })
      },

      deleteContact(Contact) {
        axios.delete(`https://localhost:44357/api/Contacts/DeleteContactById?Id_contact=${Contact.Id_contact}`)
        .then((resp) => {
            this.getContactsByIdCustomer()
            this.successMessage = resp.data
            this.successAlert = true
        }).catch((err) => {
          console.log(err.response)
          this.errorMessage = err.response.data
          this.errorAlert = true
        })
      },

      editItem (item) {
        this.editedIndex = this.contacts.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialog = true
      },

      deleteItem (item) {
        this.editedIndex = this.contacts.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialogDelete = true
      },

      deleteItemConfirm () {
        this.deleteContact(this.editedItem)
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

      save () {
        if (this.editedIndex > -1) {
          this.updateContact(this.editedItem)
          
        } else {
          this.editedItem.Id_customer = this.IdCustomer
          this.sendContact(this.editedItem)
        }
        this.close()
      },
    },
  }
</script>

<style>

</style>