<template>
  <v-app>
    <div v-if="loading" class="circularLoading">
      <v-progress-circular indeterminate color="red" size="100"></v-progress-circular>
    </div>
    <v-main v-else>
      <router-view v-if="user" v-bind:us="user" @logout="logout" />
      <Login @edituser="setuser" v-else />
    </v-main>
  </v-app>
</template>

<script>
import Axios from "axios";
import Login from "./components/Login.vue";

export default {
  components: { Login },
  name: "App",
  data() {
    return {
      user: null,
      alert: "",
      loading: true,
    };
  },
  methods: {
    checkUser() {
      this.loading = true;
      const cookies = document.cookie.split(";");
      const accCookie = cookies[0].split("=");
      const accessKey = accCookie[1];
      Axios({
        method: "get",
        url: "http://localhost:4422/getuser/",
        params: {
          id: accessKey,
        },
      })
        .then((res) => {
          if (res.data.length === 0) {
            this.user = null;
          } else {
            this.user = res.data[0];
          }
          this.loading = false;
        })
        .catch((err) => {
          console.log(err);
          this.alert = "Erreur de connexion " + err;
        });
    },
    setuser(a) {
      this.user = a;
      document.cookie =
        "accessKey=" + this.user.id + ";expires=Sat, 31 Dec 2022 00:00:01 GMT";
    },
    logout(){
      document.cookie = "accessKey=; expires=01 Oct 1970 00:00:00 GMT";
      this.checkUser()
    }
  },
  beforeMount() {
    this.checkUser();
  },
};
</script>
<style lang="scss">
* {
  &::-webkit-scrollbar {
    display: none;
  }
}
.circularLoading{
  width: 100vw;
  height: 100%;
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
}
body {
  margin: 0;
  padding: 0;
  .v-application {
    font-family: var(--fontText);
  }
}
@font-face {
  font-family: "Work Sans";
  src: url("./fonts/WorkSans-Regular.ttf");
}
@font-face {
  font-family: "SF";
  src: url("./fonts/SFPRODISPLAYREGULAR.OTF");
}
:root {
  --fontText: "SF";
  --primary: #5cb85c;
}
</style>
