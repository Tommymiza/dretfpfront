<template>
  <div>
    <h1>Personnel</h1>
    <v-card>
      <v-card-title>
        Demande de congés:
        <v-spacer></v-spacer>
        <div>
          <v-chip @click="openForm">
            <v-icon>mdi-account-plus</v-icon>
            Ajout d'une demande:
          </v-chip>
        </div>
        <v-spacer></v-spacer>
        <v-text-field
          v-model="search"
          append-icon="mdi-magnify"
          label="Recherche"
          single-line
          hide-details
        ></v-text-field>
      </v-card-title>
      <v-data-table
        :headers="columnsListe"
        :items="liste"
        class="elevation-4"
        :items-per-page="5"
        :search="search"
        :loading="loading"
        loading-text="Chargement ..."
      >
        <template v-slot:[`item.status`]="{ item }">
          <v-chip :color="getColor(item.status)" dark>
            {{ item.status }}
          </v-chip>
        </template>
      </v-data-table>
    </v-card>
    <div id="ajoutabsence">
      <v-dialog v-model="open" max-width="400">
        <v-card>
          <v-card-title>Demande d'absence:</v-card-title>
          <v-form ref="form" lazy-validation @submit.prevent="addabsence">
            <v-textarea
              label="Motif: "
              required
              outlined
              type="textarea"
              :rules="[rules.required]"
              v-model="motif"
            >
            </v-textarea>
            <v-text-field
              label="date de début:"
              type="date"
              outlined
              :rules="[rules.required]"
              v-model="datedeb"
            ></v-text-field>
            <v-text-field
              type="number"
              label="Nombre de jours: "
              outlined
              :rules="[rules.min, rules.max]"
              v-model="nbJour"
            ></v-text-field>
            <div>
              <v-btn :loading="loadingbtn" type="submit" color="blue"
                ><v-icon>mdi-account-plus</v-icon>Ajouter</v-btn
              >
              <v-btn @click="() => this.$refs.form.reset()" color="red"
                ><v-icon>mdi-cancel</v-icon>Reset</v-btn
              >
            </div>
          </v-form>
        </v-card>
      </v-dialog>
    </div>
  </div>
</template>

<script>
import Axios from "axios";

export default {
  name: "Dashboard",
  props: ["utilisateur"],
  data() {
    return {
      open: false,
      rules: {
        min: (v) => v > 0 || "Le nombre doit être n > 0",
        required: (value) => !!value,
        max: (v) => v <= this.$props.utilisateur.nbConge || `Il ne vous reste que ${this.$props.utilisateur.nbConge} jours de conge`
      },
      columnsListe: [
        {
          text: "Nom",
          value: "nom",
        },
        {
          text: "Motif",
          value: "motif",
        },
        {
          text: "Date de début",
          value: "dateDeb",
        },
        {
          text: "Date fin",
          value: "dateFin",
        },
        {
          text: "Status",
          value: "status",
        },
      ],
      liste: [],
      search: "",
      selectedItem: null,
      loading: true,
      motif: "",
      datedeb: "",
      nbJour: "",
      loadingbtn: false,
    };
  },
  methods: {
    openForm() {
      this.open = true;
    },
    check() {
      this.loading = true;
      Axios({
        method: "get",
        url: "http://localhost:4422/getlistbyid/",
        params: {
          id: this.$props.utilisateur.id,
        },
      })
        .then((res) => {
          this.liste = res.data;
          this.loading = false;
        })
        .catch((err) => {
          alert(err);
          this.loading = false;
        });
    },
    addabsence() {
      if(!this.$refs.form.validate()){
        return;
      }
      this.loadingbtn = true;
      var temp = this.datedeb.split('-')
      var tempDate = new Date(temp[0],temp[1] - 1,temp[2]).getTime()
      var tempdatefin = new Date(tempDate + 86400000 * this.nbJour)
      var month = tempdatefin.getMonth() + 1
      var datefin = tempdatefin.getFullYear() + "-" + month + "-" + tempdatefin.getDate();
      this.loadingbtn = false;
      Axios({
        url: "http://localhost:4422/addabs/",
        method: "post",
        data:{
          id: this.$props.utilisateur.id,
          motif: this.motif,
          dateDeb: this.datedeb,
          dateFin: datefin,
        }
      }).then(res=>{
        if(res.data.errno){
          alert(res.data.errno)
          this.loadingbtn = false
        }else{
          this.liste = res.data
          this.open = false
        }
      }).catch(err=>{
        alert(err)
      })
    },
    getColor(item) {
      if (item === "En attente") {
        return "blue";
      } else if (item === "Acceptée") {
        return "green";
      } else {
        return "red";
      }
    },
  },
  beforeMount() {
    this.check();
  },
};
</script>

<style lang="scss" scoped>
#ajoutabsence {
  background: white;
}
form {
  padding: 20px;
  div {
    width: 100%;
    display: flex;
    flex-direction: row;
    justify-content: space-around;
  }
  input {
    font-family: var(--fontText);
  }
}
</style>
