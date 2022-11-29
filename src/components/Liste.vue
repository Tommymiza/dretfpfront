<template>
  <div>
    <h1>Responsable du personnel</h1>
    <v-card>
      <v-card-title>
        Demande de congés:
        <v-spacer></v-spacer>
        <div>
          <v-chip @click="openForm">
            <v-icon>mdi-account-plus</v-icon>
            Ajout d'un personnel
          </v-chip>
          <v-chip @click="openPersonnel">
            <v-icon>mdi-eye</v-icon>
            Voir liste
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
          <v-chip :color="getColor(item.status)" dark @click="change(item)">
            {{ item.status }}
          </v-chip>
        </template>
      </v-data-table>
    </v-card>
    <div v-if="selectedItem">
      <v-dialog v-model="open" max-width="400">
        <div class="dialog">
          <h3>Nom du personnel: {{ selectedItem.nom }}</h3>
          <h3>Motif:</h3>
          <p>{{ selectedItem.motif }}</p>
          <h3>Date:</h3>
          <p>{{ selectedItem.dateDeb }} - {{ selectedItem.dateFin }}</p>
          <div>
            <v-btn
              class="ma-2"
              color="green"
              dark
              @click="send(true)"
              :loading="loadingBtn"
            >
              Accept
              <v-icon dark right> mdi-checkbox-marked-circle </v-icon>
            </v-btn>
            <v-btn
              class="ma-2"
              color="red"
              dark
              @click="send(false)"
              :loading="loadingBtn1"
            >
              Decline
              <v-icon dark right> mdi-cancel </v-icon>
            </v-btn>
          </div>
        </div>
      </v-dialog>
    </div>
    <div>
      <v-dialog v-model="showPersonnel" max-width="800">
        <v-card>
          <v-card-title>
            Personnels:
            <v-spacer></v-spacer>
            <v-text-field
              v-model="search1"
              append-icon="mdi-magnify"
              label="Recherche"
              single-line
              hide-details
            ></v-text-field>
          </v-card-title>
          <v-data-table
            :headers="columns"
            :items="personnel"
            class="elevation-4"
            :items-per-page="5"
            :search="search1"
            :loading="loading1"
            loading-text="Chargement ..."
          >
            <template v-slot:[`item.status`]="{ item }">
              <v-chip :color="getColor1(item.status)" dark>
                {{ item.status }}
              </v-chip>
            </template>
          </v-data-table>
        </v-card>
      </v-dialog>
      <v-dialog v-model="showForm" max-width="500">
        <v-card>
          <v-card-title>
            Ajout d'un personnel:
          </v-card-title>
          <v-form @submit.prevent="addPersonnel" lazy-validation ref="form">
            <v-text-field
              label="Username:"
              prepend-inner-icon="mdi-account-circle"
              outlined
              aria-required
              v-model="username"
              :rules="required"
              required
            />
            <v-select
              :items="fonction"
              label="Fonction"
              :rules="required"
              outlined
              v-model="fonct"
            ></v-select>
            <v-text-field
              label="Password:"
              v-model="password"
              type="password"
              prepend-inner-icon="mdi-lock"
              outlined
              :rules="required"
              required
            />
            <div>
              <v-btn :loading="addloading" type="submit" color="blue"
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
import { v1 } from "uuid";

export default {
  props: ["utilisateur"],
  name: "Liste",
  data() {
    return {
      columns: [
        {
          text: "Nom",
          value: "nom",
        },
        {
          text: "Fonction",
          value: "fonction",
        },
        {
          text: "Jours de congés",
          value: "nbConge",
        },
        {
          text: "Status",
          value: "status",
        },
      ],
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
      loading: false,
      loading1: false,
      loadingBtn: false,
      loadingBtn1: false,
      addloading: false,
      open: false,
      liste: [],
      selectedItem: null,
      search: "",
      search1: "",
      personnel: [],
      showPersonnel: false,
      showForm: false,
      username: "",
      password: "",
      fonction: ["Personnel", "Responsable"],
      fonct: "",
      required: [(v) => !!v],
    };
  },
  methods: {
    getList() {
      this.loading = true;
      Axios({
        method: "get",
        url: "http://localhost:4422/getlist/",
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
    change(i) {
      if (i.status === "En attente") {
        this.selectedItem = i;
        this.open = true;
      }
    },
    getColor1(item) {
      if (item === "Présent") {
        return "green";
      } else {
        return "red";
      }
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
    formatDate(date) {
      var temp = date.split("/");
      return new Date(temp[2], temp[1], temp[0]);
    },
    send(stat) {
      if (stat) {
        this.loadingBtn = true;
      } else {
        this.loadingBtn1 = true;
      }
      var diff =
        this.formatDate(this.selectedItem.dateFin) -
        this.formatDate(this.selectedItem.dateDeb);
      diff = diff / 86400000;
      Axios({
        method: "post",
        url: "http://localhost:4422/change/",
        data: {
          idConge: this.selectedItem.idConge,
          status: stat ? 2 : 0,
          nbJour: this.selectedItem.nbConge - diff,
          id: this.selectedItem.idU,
        },
      }).then((result) => {
        this.loadingBtn = false;
        this.loadingBtn1 = false;
        this.open = false;
        this.liste = result.data;
      });
    },
    openPersonnel() {
      this.loading1 = true;
      this.showPersonnel = true;
      Axios({
        method: "get",
        url: "http://localhost:4422/getpersonnel",
      })
        .then((result) => {
          this.personnel = result.data;
          this.loading1 = false;
        })
        .catch((err) => {
          alert(err);
          this.loading1 = false;
        });
    },
    openForm() {
      this.showForm = true;
    },
    addPersonnel() {
      if (!this.$refs.form.validate()) {
        return;
      }
      this.addloading = true;
      var id = v1().split("-").join("");
      Axios({
        method: "post",
        url: "http://localhost:4422/addpersonnel/",
        data: {
          id: id,
          nom: this.username,
          fonction: this.fonct,
          mdp: this.password,
        },
      })
        .then((resultat) => {
          this.$refs.form.reset();
          this.addloading = false;
          this.showForm = false;
        })
        .catch((err) => {
          alert(err);
          this.addloading = false;
        });
    },
  },
  beforeMount() {
    this.getList();
  },
};
</script>
<style lang="scss" scoped>
.table {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
}
.dialog {
  display: flex;
  flex-direction: column;
  align-items: center;
  background: white;
  padding: 20px;
  h3 {
    margin: 0;
    margin-bottom: 20px;
    align-self: flex-start;
  }
  p {
    font-size: 17px;
  }
}
h1 {
  color: var(--primary);
  margin: 40px 0px;
}
form {
  padding: 20px;
  display: flex;
  flex-direction: column;
  div{
    display: flex;
    flex-direction: row;
    justify-content: space-around;
  }
}
</style>
