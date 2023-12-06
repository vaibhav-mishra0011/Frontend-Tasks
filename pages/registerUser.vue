<template>
  <div>
    <v-snackbar v-model="snackbar" :timeout="2000">
      {{ snackbarText }}
      <template v-slot:action="{ attrs }">
        <v-btn color="blue" text v-bind="attrs" @click="snackbar = false">
          Close
        </v-btn>
      </template>
    </v-snackbar>

    <div class="register-container">
      <label for="username">User name:</label>
      <input type="text" v-model="user.name" required />

      <label for="username">Email Address:</label>
      <input type="email" v-model="user.email" required />

      <label for="password">Password:</label>
      <input type="password" v-model="user.password" required />

      <button color="primary" @click="register" :loading="loading">
        Register
      </button>
      <h4 class="mt-5 text-center">
        if already Register <nuxt-link to="/login"> login here</nuxt-link>
      </h4>
    </div>
  </div>
</template>

  <script>
export default {
  auth: false,
  data() {
    return {
      loading: false,
      user: {
        name: "",
        email: "",
        password: "",
        role_id: 2,
      },
    };
  },
  methods: {
    async register() {
      try {
        this.loading = true;
        await this.$axios.get("/sanctum/csrf-cookie");
        const res = await this.$axios.post("/api/register", this.user);
        if (res.data.status == 200) {
          this.$router.push("/login");
          console.log(res, "res");
          this.snackbarText = "Registered successfully";
          this.snackbar = true;
        }
      } catch (e) {
        console.log(e);
      } finally {
        this.loading = false;
      }
    },
  },
};
</script>
<style scoped>
.register-container {
  max-width: 300px;
  margin: 0 auto;
  padding: 20px;
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

label {
  display: block;
  margin-bottom: 8px;
}

input {
  width: 100%;
  padding: 8px;
  margin-bottom: 16px;
  box-sizing: border-box;
}

button {
  background-color: #4caf50;
  color: #fff;
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
</style>