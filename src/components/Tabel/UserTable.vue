<template>
  <v-data-table
    :headers="headers"
    :items="users"
    :sort-by="[{ key: 'nama', order: 'asc' }]"
    :header-props="{ class: 'text-md !font-semibold !text-gray-700' }"
    class="border rounded-md shadow-lg flex items-stretch"
  >
    <template v-slot:top>
      <v-toolbar flat :elevation="1" color="amber-lighten-3">
        <!-- <v-toolbar-title>My CRUD</v-toolbar-title> -->
        <v-divider class="mx-4" inset vertical></v-divider>
        <v-spacer></v-spacer>
        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ props }">
            <v-btn
              class="my-5 mr-5"
              color="deep-orange-darken-1"
              rounded="2"
              variant="outlined"
              v-bind="props"
            >
              New Item
            </v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="text-h5">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-col cols="12" md="4" sm="6">
                    <v-text-field
                      v-model="editedItem.nama"
                      label="Nama Lengkap"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12" md="4" sm="6">
                    <v-text-field
                      v-model="editedItem.email"
                      label="Email"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12" md="4" sm="6">
                    <v-text-field
                      v-model="editedItem.usia"
                      label="Usia"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12" md="4" sm="6">
                    <p>Status Keanggotaan</p>
                    <v-switch
                      v-model="editedItem.status"
                      color="primary"
                      label="Aktif"
                    ></v-switch>
                  </v-col>
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue-darken-1" variant="text" @click="close">
                Cancel
              </v-btn>
              <v-btn color="blue-darken-1" variant="text" @click="save">
                Save
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-dialog v-model="dialogDelete" max-width="500px">
          <v-card>
            <v-card-title class="text-h5"
              >Are you sure you want to delete this item?</v-card-title
            >
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue-darken-1" variant="text" @click="closeDelete"
                >Cancel</v-btn
              >
              <v-btn
                color="blue-darken-1"
                variant="text"
                @click="deleteItemConfirm"
                >OK</v-btn
              >
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>

    <template v-slot:item.status="{ item }">
      <div class="text-center">
        <v-chip
          :color="item.status ? 'green' : 'red'"
          :text="item.status ? 'Aktif' : 'Tidak Aktif'"
          class="text-uppercase"
          size="small"
          label
        ></v-chip>
      </div>
    </template>

    <template v-slot:item.actions="{ item }">
      <v-icon class="me-2" color="success" size="large" @click="editItem(item)">
        mdi-pencil
      </v-icon>
      <v-icon size="large" color="warning" @click="deleteItem(item)">
        mdi-delete
      </v-icon>
    </template>
    <template v-slot:no-data>
      <p>No Data Available</p>
    </template>
  </v-data-table>
</template>

<script>
import axios from "axios";
export default {
  data: () => ({
    dialog: false,
    dialogDelete: false,
    headers: [
      {
        align: "center",
        key: "nama",
        title: "Nama Lengkap",
      },
      { key: "email", title: "Alamat Email", align: "center", sortable: false },
      { key: "usia", title: "Usia", align: "center" },
      { key: "status", title: "Status Keanggotaan", align: "center" },
      { title: "Actions", key: "actions", sortable: false },
    ],
    users: [],
    editedIndex: -1,
    editedItem: {
      nama: "",
      email: "",
      usia: 0,
      status: 0,
    },
    defaultItem: {
      nama: "",
      email: "",
      usia: 0,
      status: 0,
    },
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "New Item" : "Edit Item";
    },
  },

  watch: {
    dialog(val) {
      val || this.close();
    },
    dialogDelete(val) {
      val || this.closeDelete();
    },
  },

  created() {
    this.initialize();
  },

  methods: {
    async initialize() {
      try {
        // Fetch Data Dari Endpoint API
        const res = await axios.get("https://dummyjson.com/users");
        this.users = res.data.users.map((user) => ({
          nama: `${user.firstName} ${user.lastName}`,
          email: user.email,
          usia: user.age,
          role: user.role,
          status: user.role === "moderator" ? false : true,
        }));
      } catch (e) {
        console.error("Gagal mengambil data: ", e);
      }
    },

    editItem(item) {
      this.editedIndex = this.users.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      this.editedIndex = this.users.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },

    deleteItemConfirm() {
      this.users.splice(this.editedIndex, 1);
      this.closeDelete();
    },

    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    closeDelete() {
      this.dialogDelete = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    save() {
      if (this.editedIndex > -1) {
        Object.assign(this.users[this.editedIndex], this.editedItem);
      } else {
        this.users.push(this.editedItem);
      }
      this.close();
    },
  },
};
</script>
