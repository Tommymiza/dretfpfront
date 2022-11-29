<template>
  <div id="login">
    <v-form @submit.prevent="login" lazy-validation ref="form">
      <v-text-field
        label="Username:"
        prepend-inner-icon="mdi-account-circle"
        outlined
        aria-required
        v-model="username"
        :rules="required"
        required
      />
      <v-text-field
        label="Password:"
        v-model="password"
        type="password"
        prepend-inner-icon="mdi-lock"
        outlined
        :rules="required"
        required
      />
      <v-btn :loading="loginloading" type="submit"><v-icon>mdi-lock-open</v-icon>Login</v-btn>
    </v-form>
  </div>
</template>

<script>
import Axios from "axios";

export default {
  data() {
    return {
      username: "",
      password: "",
      required: [(v) => !!v || "Veuillez remplir les champs"],
      loginloading: false
    };
  },
  methods: {
    login() {
      if (!this.$refs.form.validate()) {
        return;
      }
      this.loginloading = true
      Axios({
        method: "post",
        url: "http://localhost:4422/login/",
        data: {
          username: this.username,
          password: this.password,
        },
      }).then((res) => {
        if (res.data.status) {
          alert(res.data.status);
        } else {
          this.$emit("edituser", res.data);
        }
        this.loginloading = false
      }).catch(err=>{
        this.loginloading = false
        alert(err)
      })
    },
  },
};
</script>

<style lang="scss" scoped>
form {
  display: flex;
  flex-direction: column;
  align-items: center;
  .v-text-field ::v-deep input {
    font-size: 20px;
  }
  .v-text-field ::v-deep label {
    font-size: 20px;
  }
}
#login {
  display: flex;
  flex-direction: row;
  width: 100vw;
  height: 100vh;
  justify-content: center;
  align-items: center;
}
</style>
