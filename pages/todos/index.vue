<template>
  <div>
    <v-container>
      <v-row v-if="isLoadingData">
        <v-col>
          <v-row align="center" justify="center">
            <v-col cols="5">
              <v-progress-circular
                :size="100"
                :width="10"
                color="#FE696E"
                indeterminate
                v-bind:style="{ margin: '50%' }"
              ></v-progress-circular>
            </v-col>
          </v-row>
        </v-col>
      </v-row>
      <v-row v-else>
        <v-col cols="1" class="mt-10">
          <v-icon
            style="backgroundColor:rgba(238, 238, 238, 1; borderRadius:50%"
            size="60px"
            class="pa-2"
            color="#F94249"
          >
            mdi-animation
          </v-icon>
        </v-col>
        <v-col cols="9" class="mt-10">
          <h1>Todo List</h1>
          <p>Manage Task List</p>
        </v-col>
      </v-row>
      <v-row dense no-gutters>
        <v-spacer></v-spacer>
        <v-col cols="2">
          <NuxtLink to="/todos/create">
            <v-btn
              @click="gotoCreate()"
              width="100%"
              dark
              color="#F94249"
              class="my-2"
            >
              <v-icon>mdi-plus-box</v-icon> Add New Todo
            </v-btn></NuxtLink
          >
        </v-col>
      </v-row>
      <v-card>
        <v-card-title>
          <v-row align="center" justify="end">
            <v-col cols="6">
              <v-text-field
                v-model="search"
                append-icon="mdi-magnify"
                label="Search"
                single-line
                hide-details
              ></v-text-field>
            </v-col>
          </v-row>
        </v-card-title>
      </v-card>
      <v-data-table :headers="headers" :items="todos" :search="search">
        <template #item="{ item }">
          <tr>
            <td>{{ item.title }}</td>
             <td>
          <div class="description-cell" :title="item.description">
            {{ truncateDescription(item.description, 20) }}
          </div>
        </td>
            <td>{{ item.priority }}</td>
            <td>{{ item.status }}</td>
            <td>{{ item.created_at.substring(0, 10) }}</td>
            <td>
                              <v-tooltip bottom>
                  <template v-slot:activator="{ on, attrs }">
                  <NuxtLink
                    :to="{
                      path: `/todos/${item.id}`,
                      query: { mode: 'view' },
                    }"
                    style="color: black"
                  >
                    <v-btn small icon v-bind="attrs" v-on="on">
                      <v-icon small>mdi-eye</v-icon>
                    </v-btn>
                  </NuxtLink>
                </template>
                <span>View</span>
              </v-tooltip>
              &nbsp;
              <v-tooltip bottom>
                  <template v-slot:activator="{ on, attrs }">
                  <NuxtLink
                    :to="{
                      path: `/todos/${item.id}`,
                      query: { mode: 'update' },
                    }"
                    style="color: black"
                  >
                    <v-btn small icon v-bind="attrs" v-on="on">
                      <v-icon small>mdi-pencil</v-icon>
                    </v-btn>
                  </NuxtLink>
                </template>
                <span>Edit</span>
              </v-tooltip>
              &nbsp;
              <v-tooltip bottom>
                  <template v-slot:activator="{ on, attrs }">
                  <v-btn
                    small
                    icon
                    v-bind="attrs"
                    v-on="on"
                    @click="remove(item.id)"
                  >
                    <v-icon small>mdi-trash-can</v-icon>
                  </v-btn>
                </template>
                <span>Delete</span>
              </v-tooltip>
               <template>
                <v-dialog
                  :retain-focus="false"
                  v-model="deleteDialog"
                  max-width="600px"
                >
                  <v-card>
                    <v-card-title>
                      <span class="headline"
                        >Are you sure you want to delete this todo?</span
                      >
                    </v-card-title>
                    <v-card-actions>
                      <v-spacer></v-spacer>
                      <v-btn
                        color="red darken-1"
                        text
                        @click="
                          deleteTodo(deleteItem.code);
                          deleteDialog = false;
                        "
                      >
                        Delete
                      </v-btn>
                      <v-btn
                        color="blue darken-1"
                        text
                        @click="deleteDialog = false"
                      >
                        Cancel
                      </v-btn>
                    </v-card-actions>
                  </v-card>
                </v-dialog>
              </template>
            </td>
          </tr>
        </template>
      </v-data-table>
      
    </v-container>
    <v-snackbar v-model="confirmaionSnackbar" :timeout="3000">
      <v-icon color="green">mdi-check-bold</v-icon>
      {{ responsesuccessMsg }}
    </v-snackbar>
    <v-snackbar v-model="errorSnackBar" :timeout="3000">
      <v-icon color="red">mdi-close</v-icon>
      {{ responseErrorMsg }}
    </v-snackbar>
    <v-snackbar v-model="validationErrorSnackbar" :timeout="3000">
      <v-icon color="red">mdi-close</v-icon>
      {{ validationErrorMessage }}
    </v-snackbar>
  </div>
</template>


<script>
import axios from "axios";
export default {
  name: "todos-update",
  data() {
    return {
      headers: [
        { key: "title", title: "Title", sortable: false },
        { key: "description", title: "Description", sortable: false },
        { key: "priority", title: "Priority" },
        { key: "status", title: "Status" },
        { key: "created_at", title: "Created Date" },
        {
          key: "Action",
          title: "Actions",
          divider: true,
          sortable: false,
        },
      ],
      validationErrorMessage: "",
      confirmaionSnackbar: false,
      validationErrorSnackbar: false,
      errorSnackBar: false,
      responsesuccessMsg: "",
      responseErrorMsg: "",
      isLoadingData: true,
      deleteDialog: false,
      deleteItem: [],
      search: "",
      todos: [],
    };
  },
  async mounted() {
    this.getAllRecords();
  },
    watch: {
    search(search) {
      this.getAllRecords();
    },
  },
  methods: {
    remove(code) {
      this.deleteItem.code = code;
      this.deleteDialog = true;
    },

async deleteTodo(id) {
  try {
    const res = await axios.delete(`${this.$config.public.baseURL}/todos/${id}`, {
      headers: {
        "Content-Type": "application/json",
      },
    });


    if (res.status === 200) {
      this.responsesuccessMsg = res.data.message;
      this.confirmaionSnackbar = true;
      setTimeout(() => {
        this.getAllRecords();
      }, 1000);
    } else {
      this.responsesuccessMsg = res.data.message;
      this.errorSnackBar = true;
    }
  } catch (error) {
    console.error('Error deleting todo:', error);
    this.responsesuccessMsg = "An error occurred while deleting the todo.";
    this.errorSnackBar = true;
  }
},
 truncateDescription(description, maxLength) {
      if (description.length > maxLength) {
        return description.substring(0, maxLength) + '...';
      }
      return description;
    },

    async getAllRecords() {
      const response = await axios.get(`${this.$config.public.baseURL}/todos`, {
         params: {
      search: this.search,
    },
        headers: {
          "Content-Type": "application/json",
        },
      });
      if (response.status == 200) {
        this.todos = response?.data?.data;
        this.isLoadingData = false;
        this.responsesuccessMsg = response.data.message;
      } else {
        this.todos = [];
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.delete-dialog {
    background-color: transparent !important;
}
.description-cell {
  max-width: 150px; /* Adjust as needed */
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  cursor: pointer;
}
</style>