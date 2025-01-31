<template>
  <v-container class="mt-16" id="container">
    <v-snackbar v-model="snackbar" :timeout="timeout" top>
      {{ text }}

      <template v-slot:action="{ attrs }">
        <v-btn color="blue" text v-bind="attrs" @click="snackbar = false">
          Close
        </v-btn>
      </template>
    </v-snackbar>
    <v-card id="register-form" color="grey darken-4" class="pa-8">
      <h1 class="white--text">Register</h1>
      <form @submit.prevent="login">
        <v-text-field
          label="Email"
          hide-details="auto"
          :rules="emailRules"
          v-model="email"
          name="username"
          dark
        ></v-text-field>
        <v-text-field
          dark
          label="Password"
          :rules="passwordRules"
          hide-details="auto"
          type="password"
          v-model="password"
          name="password"
        ></v-text-field>
        <v-text-field
          dark
          label="Confirm Password"
          hide-details="auto"
          :rules="[passwordConfirmationRule]"
          type="password"
          ref="password"
          v-model="confirmPassword"
        ></v-text-field>
        <v-btn class="mt-10 orange darken-2 accent-2 white--text" type="submit">
          submit
        </v-btn>
      </form>
    </v-card>
    <v-progress-linear
      id="progress-bar"
      color="orange"
      indeterminate
      height="6"
      bottom
      v-if="loading"
    ></v-progress-linear>
  </v-container>
</template>

<script>
import { mapMutations } from "vuex";
export default {
  data: () => {
    return {
      loading: false,
      snackbar: false,
      text: "The email address is already registered",
      timeout: 5000,
      email: "",
      password: "",
      confirmPassword: "",
      passwordRules: [
        (v) => !!v || "Password is required",
        (v) => v.length <= 50 || "Max 50 characters",
        (v) => v.length >= 8 || "Min 8 characters",
      ],
      emailRules: [
        (v) => !!v || "Email is required",
        (v) => /.+@.+/.test(v) || "E-mail must be valid",
      ],
    };
  },
  computed: {
    passwordConfirmationRule: function () {
      return () =>
        this.password === this.confirmPassword || "Password must match";
    },
  },
  methods: {
    ...mapMutations(["setToken"]),
    login: function (e) {
      let emailCount = 0;
      let passwordCount = 0;
      this.emailRules.forEach((rule) => {
        if (rule(this.email) === true) {
          emailCount++;
        }
      });
      this.passwordRules.forEach((rule) => {
        if (rule(this.password) === true) {
          passwordCount++;
        }
      });
      if (
        passwordCount === this.passwordRules.length &&
        emailCount == this.emailRules.length &&
        this.passwordConfirmationRule() === true
      ) {
        const data = new FormData(e.target);
        const value = Object.fromEntries(data.entries());
        this.loading = true;
        fetch(process.env.VUE_APP_BACKEND_SERVICE_URL + "register", {
          method: "POST",
          mode: "cors",
          headers: {
            "Content-Type": "application/json",
            "Access-Control-Allow-Origin": "*",
          },
          body: JSON.stringify(value),
        })
          .then((res) => {
            return res.json();
          })
          .then((data) => {
            this.loading = false;
            if (data.token === undefined) {
              this.snackbar = true;
            } else {
              this.$store.commit("setToken", data.token);
              this.$router.push("/");
              window.location.reload();
            }
          });
      }
    },
  },
};
</script>

<style lang="scss" scoped>
#progress-bar {
  border-radius: 0 0 4px 4px;
}
#register-form {
  border-radius: 4px 4px 0 0;
}
@media screen and (min-width: 600px) {
  #container {
    width: 60%;
  }
}
@media screen and (min-width: 1200px) {
  #container {
    width: 40%;
  }
}
</style>
