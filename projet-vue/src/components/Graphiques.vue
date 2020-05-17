<template>
  <div class="hello">
    <h1> Bienvenue sur les graphiques </h1>
      <div id="Graphique">
        <div class="contentGraphique">
          <p>Voici votre page Graphiques.
            Ici vous avez accès à la température et à la luminosité en temps réel.
          </p>
        </div>
      </div>
    <h1>{{ msg }}</h1>
    <div id="app">
       
      <Chart  :k="componentKey" :which_esps= "which_esps"/>
      <label for="MAC">Votre adresse MAC : </label>
      <input type="text" id="MAC" name="MAC"><br><br>
      <md-button class="md-accent md-raised" type="button" v-on:click="ajoutMAC()">Rejoindre en tant qu'invité</md-button>
    </div>
    <!-- <md-input v-model="name"></md-input>
    <md-button class="md-primary md-raised" v-on:click="addFloat(name)">Add to fleet</md-button>-->
      
    <h1>LED states</h1>
    <md-table>
      <md-table-row slot="md-table-row">
        <md-table-head md-label="Name">ESP</md-table-head>
        <md-table-head md-label="Actions">Action</md-table-head>
      </md-table-row>
      <md-table-row>
        <md-table-cell md-label="Name">Piscine</md-table-cell>
        <md-table-cell md-label="Actions">
          <md-button class="md-primary md-raised" v-on:click="switchState('30:AE:A4:86:CA:7C')">PING</md-button>
        </md-table-cell>
      </md-table-row>
      <md-table-row>
        <md-table-cell md-label="Name">Poulailler</md-table-cell>
        <md-table-cell md-label="Actions">
          <md-button class="md-primary md-raised" v-on:click="switchState('30:AE:A4:86:C3:20')">PING</md-button>
        </md-table-cell>
      </md-table-row>
      <md-table-row>
        <md-table-cell md-label="Name">{{ which_esps[3] }}</md-table-cell>
        <md-table-cell md-label="Actions">
          <md-button class="md-primary md-raised" v-on:click="switchState(which_esps[3])">PING</md-button>
        </md-table-cell>
      </md-table-row>
    </md-table>
  </div>
</template>

<script>
//var Highcharts = require("highcharts");
import Chart from "./Chart";

//import highcharts from 'https://code.highcharts.com/highcharts.js'
export default {
  name: "Graphiques",
  data() {
    return {
      states: [],
      node_url: "http://51.210.15.67:3000",
      which_esps: [
        "30:AE:A4:86:C3:20",
        "30:AE:A4:86:CA:7C",
        ""
      ],
      name: "",
      componentKey: 0
    };
  },
  components: {
    Chart
  },
  props: {
    msg: String
  },
  mounted() {},

  methods: {
    ajoutMAC() {
      var adMac = document.getElementById("MAC");
      console.log("MAC: " + adMac.value);
      this.which_esps[3] = adMac.value;
      console.log(this.which_esps);
     // this.start();
      this.forceRerender();
    },
     forceRerender() {
      this.componentKey += 1;  
    },
    start: function() {
      for (var i = 0; i < this.which_esps.length; i++) {
        this.getStates("/esp/light", [], this.which_esps[i]);
      }
      for (var j = 0; j < this.which_esps.length; j++) {
        this.getStates("/esp/temp", [], this.which_esps[j]);
      }
    },
    getStates(path_on_node, serie, wh) {
      this.node_url = "http://51.210.15.67:3000";
      //console.log("Test fonction getStates");
      var liste = [];
      let url = this.node_url + path_on_node + "?who=" + wh;
      fetch(url)
        .then(responseJSON => {
          return responseJSON.json();
        })
        .then(responseJS => {
          this.items = responseJS;

          if (this.items) {
            this.items.forEach(function(element) {
              liste.push([Date.parse(element.date), element.value]);
            });
          }
          serie.data = liste;
          //console.log(liste);
        });
    },

    async switchState(wh) {
      var url = this.node_url + "/esp/led";
      var message = "on";
      var header = new Headers();
      header.append("Content-Type", "application/json");
      await fetch(url, {
        method: "POST",
        headers: header,
        body: JSON.stringify({
          message: message,
          who: wh
        })
      });
    },

    async addFlotte(wh) {
      this.which_esps.append(wh);
      var url = this.node_url + "/flotte";
      var header = new Headers();
      header.append("Content-Type", "application/json");
      await fetch(url, {
        method: "POST",
        headers: header,
        body: JSON.stringify({
          who: wh
        })
      });
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.md-table-head {
  text-align: center;
}
</style>
