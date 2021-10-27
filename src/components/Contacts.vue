<template>
  <v-data-table
    :headers="headers"
    :items="contacts"
    sort-by="calories"
    class="elevation-1"
  >
    <template v-slot:top>
      <v-toolbar
        flat
      >
        <v-toolbar-title>Contacts</v-toolbar-title>
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
              Add New
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
                  >
                    <v-text-field
                      v-model="editedItem.name"
                      label="Name"
                    ></v-text-field>
                  </v-col>
                </v-row>

                <v-row>
                  <v-col
                    cols="12"
                  >
                    <v-text-field
                      v-model="editedItem.cellphoneNumber"
                      label="Cellphone Number"
                    ></v-text-field>
                  </v-col>

                </v-row>
                <v-row>
                  <v-col
                    cols="12"
                  >
                    <v-text-field
                      v-model="editedItem.telephoneNumber"
                      label="Telephone Number"
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
                Cancel
              </v-btn>
              <v-btn
                color="blue darken-1"
                text
                @click="save"
              >
                Save
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-dialog v-model="dialogDelete" max-width="500px">
          <v-card>
            <v-card-title class="text-h5">Delete contact?</v-card-title>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="closeDelete">Cancel</v-btn>
              <v-btn color="blue darken-1" text @click="deleteItemConfirm">OK</v-btn>
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
        @click="editItem(item)"
      >
        mdi-pencil
      </v-icon>
      <v-icon
        small
        @click="deleteItem(item)"
      >
        mdi-delete
      </v-icon>
    </template>
    <template v-slot:no-data>
      <v-btn
        color="primary"
        @click="load"
      >
        Reset
      </v-btn>
    </template>
    
  </v-data-table>
</template>

<script>

  import axios from 'axios';

  let baseurl = process.env.NODE_ENV == 'development' ? 'http://localhost:5001/contact-ff327/us-central1/app': window.location.origin

  console.log( process.env.NODE_ENV )
  export default {


    data: () => ({
      dialog: false,
      dialogDelete: false,
      contact: {
        name: "",
        telephoneNumber: "",
        cellphoneNumber:""
      },

      contacts: [],
      headers: [
          
          {
            text: 'Name',
            align: 'start',
            sortable: false,
            value: 'name',
          },
          
          { text: 'Cellphone Number', value: 'cellphoneNumber' },
          { text: 'Telephone Number', value: 'telephoneNumber' },
          { text: 'Actions', value: 'actions', sortable: false },
        ],  
      editedIndex: -1,
      editedItem: {
        name: "",
        telephoneNumber: "",
        cellphoneNumber:""
      },
      defaultItem: {
        name: "",
        telephoneNumber: "",
        cellphoneNumber:""
      }, 
    }),

    methods: {
      async load(){
        try {
          let response = await axios.get(`${baseurl}/api/contacts`)
          this.contacts = response.data;  
        } catch (error) {
          alert( error ) 
        }
        
      },

      async save(){
        try {

          if (this.editedIndex > -1) {
          
            await axios.put(`${baseurl}/api/contacts/${this.editedItem.id}`, this.editedItem);
          } else {
        
            await axios.post(`${baseurl}/api/contacts`, this.editedItem);
          }

        
          this.load() 
          this.close()
        } catch (error) {
          alert( error )   
        }

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

      async deleteItemConfirm () {
        try {
          await axios.delete(`${baseurl}/api/contacts/${this.editedItem.id}`);  
          this.closeDelete()
          this.load()
        } catch (error) {
          console.log( error )
        }
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

    },

    computed: {
      formTitle () {
        return this.editedIndex === -1 ? 'New Contact' : 'Edit Contact'
      },
    },

    watch: {
      dialog (val) {
        val || this.close()
      },
      dialogDelete (val) {
        val || this.closeDelete()
      },
    },

    mounted(){
      this.load()
    }
  }
</script>
