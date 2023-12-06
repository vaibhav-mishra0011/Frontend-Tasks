<template>
  <div>
    <v-app-bar app>
      <v-toolbar-title>Hello {{ this.$auth.user.name }}</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn class="primary" @click.native.stop="addTodoDialog = true" v-if="role_id!=1">
        +Add Task
      </v-btn>
      <v-btn class="primary ml-3" @click="logout">Logout</v-btn>
    </v-app-bar>

    <v-layout row justify-end>
      <!-- confirm dialog -->
      <!-- dialog for add a new task -->
      <v-dialog v-model="addTodoDialog" max-width="290">
        <v-card class="p-5">
          <v-card-title>Add a To Do Task</v-card-title>
          <v-card-text class="pa-2">
            <v-text-field
              v-model="task.name"
              label="Enter Title"
            ></v-text-field>
            <v-text-field
              v-model="task.description"
              label="Enter Description"
            ></v-text-field>
            <v-checkbox
              class="label-set"
              height="50px"
              type="checkbox"
              v-model="isCompleted"
              label="Mark as completed"
            ></v-checkbox>
          </v-card-text>
          <v-card-actions>
            <v-btn @click="saveTask">Save</v-btn>
            <v-btn @click="handleClose">close</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <!-- dialog for edit a task -->
      <v-dialog v-model="editTodoDialog" max-width="290">
        <v-card class="p-5">
          <v-card-title>Edit Task</v-card-title>
          <v-card-text class="pa-2">
            <v-text-field
              v-model="task.name"
              label="Enter Title"
            ></v-text-field>
            <v-text-field
              v-model="task.description"
              label="Enter Description"
            ></v-text-field>
            <v-checkbox
              class="label-set"
              height="50px"
              type="checkbox"
              v-model="isCompleted"
              label="Mark as completed"
            ></v-checkbox>
          </v-card-text>
          <v-card-actions>
            <v-btn @click="updateTask" :loading="loading">Save</v-btn>
            <v-btn @click="editTodoDialog = false">close</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
      <div class="card">
        <div class="card-inner">
          <ul class="list">
            <ul class="mx-5px">
              <!-- <Task></Task> -->
              <v-container id="table">
                <v-data-table
                  :loading="loading"
                  :headers="headers"
                  :items="tasks"
                  :footer-props="{
                    'items-per-page-options': [15, 30, 50, 100, -1],
                  }"
                  class="elevation-1"
                  :server-items-length="total"
                  @update:page="handlePageUpdate"
                  @update:items-per-page="handlePerPageUpdate"
                  :sort-by.sync="sortBy"
                  :sort-desc.sync="sortDesc"
                >
                  <template #item.index="{ item, index }">
                    {{ index + 1 }}
                  </template>
                  <template #item.created_at="{ item }">
                    {{ $dayjs(item.created_at).format("YYYY/MM/DD") }}
                  </template>
                  <template #item.completed="{ item }">
                    <v-checkbox
                      v-model="item.completed"
                      @change="updateStatus(item)"
                      :disabled="checkboxDisabled"
                    >
                    </v-checkbox>
                  </template>

                  <template v-slot:item.action="{ item }">
                    <div class="d-flex">
                      <v-btn v-if="role_id!=1"
                        :loading="item.loading"
                        elevation="0"
                        icon
                        color="green"
                        @click="saveEditTask(item)"
                      >
                        <v-icon dark>mdi-pencil</v-icon>
                      </v-btn>

                      <v-btn
                        v-if="item.id!=itemIdToDelete"
                        :loading="item.loading"
                        elevation="0"
                        icon
                        color="red"
                        @click="deleteTask(item.id)"
                      >
                        <v-icon dark>mdi-delete</v-icon>
                      </v-btn>
                      
                      <v-progress-circular v-else indeterminate color="primary">
                      </v-progress-circular>

                    </div>
                  </template>
                </v-data-table>
              </v-container>
            </ul>
          </ul>
        </div>
      </div>
    </v-layout>
    <v-snackbar v-model="snackbar" :timeout="2000">
      {{ snackbarText }}
      <template v-slot:action="{ attrs }">
        <v-btn color="blue" text v-bind="attrs" @click="snackbar = false">
          Close
        </v-btn>
      </template>
    </v-snackbar>
  </div>
</template>
<script>
export default {
  data() {
    return {
      itemIdToDelete:null,
      sortBy: "created_at",
      sortDesc: true,
      editTodoId: "",
      editTodoDialog: false,
      snackbar: false,
      snackbarText: "",
      page: 1,
      perPage: 10,
      total: 0,
      isCompleted: false,
      tasks: [],
      loading: false,
      addTodoDialog: false,
      user_id: this.$auth.user.id,
      role_id: this.$auth.user.role_id,
      checkboxDisabled: this.$auth.user.role_id==1?true:false,
      task: {
        name: "",
        description: "",
        user_id: this.$auth.user.id,
      },
      headers: [
        { text: "S. No.", value: "index", sortable: false },
        { text: "Title", value: "name", sortable: true },
        { text: "Description", value: "description", sortable: true },
        { text: "completed", value: "completed", sortable: true },
        { text: "Created At", value: "created_at", sortable: true },
        { text: "Action", value: "action", sortable: false },
      ],
    };
  },
  mounted() {
    this.fetchTasks();
  },
  methods: {
    async updateStatus(item) {
      try {
        const res = await this.$axios.patch(
          `/api/task/${this.item.id}`,
          {completed:item.completed}
        );
        this.editTodoDialog = false;
        this.snackbarText = "you task has been edited";
        this.snackbar = true;
        this.fetchTasks();
      } catch (e) {
        console.log(e);
      }
      console.log(item.completed);
    },
    handlePageUpdate(page) {
      this.page = page;
      this.fetchTasks();
    },
    handlePerPageUpdate(perPage) {
      this.perPage = perPage;
      this.fetchTasks();
    },
    async logout() {
      try {
        this.snackbarText = "logged out successfully";
        this.snackbar = true;
        await this.$auth.logout();
        this.$router.push("/login");
      } catch (e) {
        console.log(e);
      }
    },
    async fetchTasks() {
      try {
        this.loading = true;
        const { data } = await this.$axios.get(
          `api/tasks?page=${this.page}&perPage=${this.perPage}&sortBy=${this.sortBy}&desc=${this.sortDesc}`
        );
        this.tasks = data.data.data;
        this.total = data.data.total;
      } catch (error) {
        console.log(error);
      } finally {
        this.loading = false;
      }
    },
    async saveTask() {
      try {
        const response = await this.$axios.post("/api/create", {
          name: this.task.name,
          description: this.task.description,
          user_id: this.user_id,
          completed: this.isCompleted,
        });
        this.snackbarText = "you task has been added";
        this.snackbar = true;
        this.fetchTasks();
        console.log(response);
      } catch (e) {
        console.log(e);
      }
      this.addTodoDialog = false;
      this.task.name = "";
      this.task.description = "";
    },
    handleClose() {
      this.addTodoDialog = false;
      console.log(this.user_id);
    },

    async deleteTask(taskId) {
      try {
        this.itemIdToDelete=taskId;
        const res = await this.$axios.delete(`/api/task/${taskId}`);
        this.snackbarText = "you task has been deleted";
        this.snackbar = true;
        this.fetchTasks();

      } catch (error) {
        console.log(error);
      } finally{

        this.itemIdToDelete=null;
      }
    },

    saveEditTask(task) {
      console.log(task);
      this.editTodoId = task.id;
      this.task.name = task.name;
      this.task.description = task.description;
      this.isCompleted = task.completed;
      this.editTodoDialog = true;
    },

    async updateTask() {

      const updatedValues = {
        name: this.task.name,
        description: this.task.description,
        completed: this.isCompleted,
      };
      console.log(updatedValues);

      try {
        this.loading=true;
        const res = await this.$axios.patch(
          `/api/task/${this.editTodoId}`,
          updatedValues
        );
        this.editTodoDialog = false;
        this.snackbarText = "you task has been edited";
        this.snackbar = true;
        this.fetchTasks();
      } catch (e) {
        console.log(e);
      }finally{
        this.loading=false;
      }
    },
  },
};
</script>

<style scoped>
#table {
  margin-top: 2rem;
  margin-right: 33rem;
  background-color: #d7d7d7;
}
#addTask {
  color: darkblue;
  background-color: beige;
}
#logout {
  margin-left: 1rem;
}
</style>
