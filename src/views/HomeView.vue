<template>
  <v-container>
    <v-btn elevation="4" id="logout" color="secondary" fab @click="()=>this.$emit('logout')">
      <v-icon>mdi-logout-variant</v-icon>
    </v-btn>
    <Liste v-if="us.fonction === 'Responsable'" :utilisateur="us" />
    <Dashboard v-else :utilisateur= "us"/>
  </v-container>
</template>

<script>
import Liste from "../components/Liste";
import Dashboard from "../components/Dashboard";

export default {
  props: ["us"],
  name: "Home",
  components: {
    Liste,
    Dashboard,
  },
};
</script>
<style scoped>
#logout{
  position: absolute;
  top: 15px;
  right: 10px;
}
</style>
