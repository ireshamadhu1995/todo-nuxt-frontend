<template>
  <div>
 
    <v-container>
       <v-row>
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
          <h1>Add New Todo</h1>
          <p>Manage Task List</p>
        </v-col>
        <v-spacer></v-spacer>
      </v-row>
      <v-card-text>
        <v-form ref="form" v-model="isValid">
          
          <v-divider></v-divider>
          <v-row class="pt-4">
            <v-col cols="6">
              <v-text-field
                dense
                outlined
                label="Title*"
                v-model="createForm.title"
                required
            :rules="[rules.required]"
              ></v-text-field>
            </v-col>
             <v-col cols="6">
                <v-select
                    label="Priority*"
                    :items="priorityOptions"
                    v-model="createForm.priority"
                    outlined
                    dense
                    required
            :rules="[rules.required, rules.priorityRule]"
                  >
                  </v-select>
            </v-col>

            <v-col cols="12">
               <v-textarea
            label="Description*"
            v-model="createForm.description"
            rows="5"
            dense
            outlined
             required
            :rules="[rules.required]"
          ></v-textarea>
            </v-col>
            <v-col cols="6">
           <v-select
                    label="Status*"
                    :items="statusOptions"
                    v-model="createForm.status"
                    outlined
                    dense
                    required
            :rules="[rules.required, rules.statusRule]"
                  >
                  </v-select>
            </v-col>
          </v-row>
          <v-row>
            <v-col>
              <v-flex class="d-flex justify-end">
                  <v-btn
                  elevation="0"
                  color="grey lighten-1"
                  dark
                  @click="goBack()"
                >
                  <v-icon>mdi-keyboard-backspace</v-icon>cancel
                </v-btn>
                <v-btn
                  elevation="0"
                  :disabled="!isValid"
                  color="blue"
                  dark
                  @click="saveTodoDetails()"
                  style="margin-left: 8px;"
                >
                  <v-icon>mdi-plus-thick</v-icon>Create
                </v-btn>
              </v-flex>
            </v-col>
          </v-row>
        </v-form>
      </v-card-text>
    </v-container>
         <v-row v-if="loading">
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
import axios from 'axios';
export default {
name: 'todosCreate',
  data() {
    return {
       rules: {
        required: (value) => !!value || "Required.",
         priorityRule: (value) => {
        if (this.priorityOptions.includes(value)) {
        return true;
      }
      return "Please select a valid value (LOW, MEDIUM, HIGH).";
        },
      statusRule: (value) => {
        if (this.statusOptions.includes(value)) {
        return true;
      }
      return "Please select a valid value (TODO, IN-PROGRESS, COMPLETED).";
        },
      },
      priorityOptions: ["LOW", "MEDIUM", "HIGH"],
      statusOptions: ['TODO', 'IN-PROGRESS', 'COMPLETED'],
      loading: false,
      validationErrorMessage: "",
      confirmaionSnackbar: false,
      validationErrorSnackbar: false,
      errorSnackBar: false,
      responsesuccessMsg: "",
      responseErrorMsg: "",
      isValid: false,
      createForm: {
        title: null,
        description: null,
        status: "TODO",
        priority: null,
      },
    };
  },

    methods: {
     goBack() {
      this.$router.back();
    },

    async saveTodoDetails() {
   try {
    this.loading = true;

    const response = await axios.post(
    `${this.$config.public.baseURL}/todos`,
          {
            title: this.createForm.title,
            description: this.createForm.description,
            priority: this.createForm.priority,
            status: this.createForm.status,
          },
          {
            headers: {
              "Content-Type": "application/json",
            },
          }
    );

    if (response.status === 200) {
      this.confirmaionSnackbar = true;
      this.responsesuccessMsg = response.data.message;
        setTimeout(() => {
        this.$router.back();
      }, 1000);
    } else {
      this.errorSnackBar = true;
      this.responseErrorMsg = response.data.message;

    }
  }catch (error) {
    this.errorSnackBar = true;
    this.responseErrorMsg = 'An error occurred during the update operation.';
  } finally {
    this.loading = false;
  }
    },
  },
}
</script> 