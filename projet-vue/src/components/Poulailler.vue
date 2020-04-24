<template>
  <div>
    <h1> Bienvenue sur le poulailler </h1>
      <div id="Poulailler">
        <div class="contentPoulailler">
          <p>Voici votre page Poulailler.
            Ici vous avez accès à la température à l'intérieur du poulailler, ainsi que la luminosité ambiante.
          </p>
          <p>
            Grâce aux boutons ci-dessous, vous pouvez fermer le poulailler de manière manuelle. Dans tous les cas, votre poulailler se ferme le soir quand la luminosité baisse, et il s'ouvre le matin au levé du soleil. Au moins les poules ne restent pas enfermé toutes la journée.
            Le deuxième bouton permet d'allumer une lampe si vous voulez allez voir le poulailler la nuit.
          </p>
        </div>
        <div class="infoPoulailler">
          <p id="temp">Température : <span id="valeur">{{lastTemp}}</span> °C</p>
          <p id="lum">Luminosité : <span id="valeur">{{lastLuminosite}}</span> Lum</p>

        </div>
      </div>

      <div class="ledPoulailler">
        <p> Gestion du poulailler </p>
        <md-button class="md-primary md-raised" v-on:click="switchState('30:AE:A4:86:C3:20')">Ouvrir le poulailler</md-button>
        <br>
        <md-button class="md-primary md-raised" v-on:click="switchState('30:AE:A4:86:C3:20')">Fermer le poulailler</md-button>
        <p>Etat du poulailler : {{etatPoulailler}}</p>

      </div>

      <div class="mangerPoulailler">
        <p> Gestion de la nourriture  </p>
          <md-button class="md-primary md-raised" v-on:click="nourrir += 1">Nourrir les poules</md-button>
        <p>Les poules ont été nourrit {{ nourrir }} fois.</p>
      </div>

      <div class="oeufPoulailler">
        <p> Gestion des œufs  </p>
          <md-button class="md-primary md-raised" v-on:click="oeufs += 1">Forcer la ponte</md-button>
        <p>Nombre d'oeufs pondus : {{ oeufs }}</p>

      </div>

      </div>
</template>

<script>

export default {
  name: "Poulailler",
  data() {
    return {
      states: [],
      lastTemp: null,
      lastLuminosite: null,
      etatPoulailler: "Fermé",
      oeufs: 0,
      nourrir: 0,
      node_url: "http://localhost:3000",
      which_esps: [
        "30:AE:A4:86:C3:20",
        "30:AE:A4:86:CA:7C",
        ""
      ],
      name: "",
      componentKey: 0
    };
  },
  created: function () {
       //this.get_states("/esp/temp", [], "");
       //this.get_states("/esp/light", [], "");
       this.process_esp();
    },
  components: {
    
  },
  props: {
    msg: String
  },
  mounted() {},

  methods: {
    start: function() {
      for (var i = 0; i < this.which_esps.length; i++) {
        this.get_states("/esp/light", [], this.which_esps[i]);
      }
      for (var j = 0; j < this.which_esps.length; j++) {
        this.get_states("/esp/temp", [], this.which_esps[j]);
      }
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
    process_esp() {
      const refreshT = 5000; // Refresh period for chart
      var esp = "30:AE:A4:86:CA:7C";
      //var esp = which_esps[i]; // L'ESP "a dessiner"

      // Gestion de la temperature
      // premier appel pour eviter de devoir attendre RefreshT
      this.get_states("/esp/temp",0, esp);
      //calls a function or evaluates an expression at specified
      //intervals (in milliseconds).
      window.setInterval(
        this.get_states,
        refreshT,
        "/esp/temp", // param 1 for get_samples()
        0, // param 2 for get_samples()
        esp
      ); // param 3 for get_samples()

      // Gestion de la lumiere
      this.get_states("/esp/light", 0, esp);
      window.setInterval(
        this.get_states,
        refreshT,
        "/esp/light", // URL to GET
        0, // Serie to fill
        esp
      ); // ESP targeted
    },
    lastValue(path, val){
       if(path == "/esp/temp"){
                      //document.getElementById("temp").innerHTML = "La Température est de " + val + "°C";
            this.lastTemp = val;
       }
       if(path == "/esp/light"){
                      //document.getElementById("light").innerHTML = "La Lumiere est de " + val + " lumen";
            this.lastLuminosite = val;
       }
       console.log("Last " + path + " : " + val);
    },

    get_states(path_on_node, serie, wh) {

      this.node_url = "http://localhost:3000";

      var liste = []
      wh = "30:AE:A4:86:CA:7C";
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
          this.lastValue(path_on_node,(liste[(liste.length - 1)][1]));

        });
    },



  }
};
</script>

<style>

h1{
  font-size: 40px;
}
#valeur{
  font-weight: bold;
}
.contentPoulailler{
  font-size:20px;
  float: left;
  width: 50%;
  padding-left: 5%;
}
.infoPoulailler{
  font-size: 20px;
  float: right;
  width: 50%;
}
#temp, #lum{
  background: #448aff;
  padding:20px;
  color:white;
  border-radius: 5px;
  margin-left: 5%;
  margin-right: 5%;
}
.ledPoulailler{
  font-size: 25px;
  margin-top: 400px;
  margin-right: 1200px;
}
.mangerPoulailler{
  font-size: 25px;
  margin-top: -165px;
}

.oeufPoulailler{
  font-size: 25px;
  margin-top: -165px;
  margin-left: 1200px;
}


.contentPoulailler{
  font-size: 18px;
  text-align: center;
}
</style>