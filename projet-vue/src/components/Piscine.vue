<template>
  <div>
    <h1> Bienvenue sur votre piscine </h1>
    <div id="Piscine">
      <div class="contentPiscine">
        <p>Voici votre page Piscine.
          Ici vous avez accès à la température de l'eau, ainsi que la luminosité ambiante.
        </p>
        <p>
          Grâce aux boutons ci-dessous, vous pouvez fermer le volet de manière manuelle. Dans tous les cas, votre volet se ferme le soir quand la luminosité baisse, et il s'ouvre le matin au levé du soleil.
          Le deuxième bouton permet d'allumer les lumières présentes aux alentours de votre piscine.
        </p>
      </div>
      <div class="infoPiscine">
        <p id="temp">Température de l'eau : <span id="valeur">{{lastTemp}}</span> °C</p>
        <p id="lum">Luminosité : <span id="valeur">{{lastLuminosite}}</span> Lum</p>

      </div>
    </div>
    <div id="boutons">
      <div class="ledPiscine">
        <p> Lumières ambiante </p>
        <md-button class="md-primary md-raised" v-on:click="switchState('30:AE:A4:86:C3:20')">Allumer la led</md-button>
        <br>
        <md-button class="md-primary md-raised" v-on:click="switchState('30:AE:A4:86:C3:20')">Eteindre la led</md-button>
      </div>
      <div class="voletPiscine">
        <p> Gestion du volet </p>
        <md-button class="md-primary md-raised" v-on:click="switchState('30:AE:A4:86:C3:20')">Ouvrir la piscine</md-button>
        <br>
        <md-button class="md-primary md-raised" v-on:click="switchState('30:AE:A4:86:C3:20')">Fermer la piscine</md-button>
      </div>
     </div> 
  </div>
</template>


<script>

export default {
  name: "Piscine",
  data() {
    return {
      states: [],
      lastTemp: null,
      lastLuminosite: null,
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
       this.test(path, val);
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
    test(path, val){
      //console.log("test " + path + " : " + val);

      if(path == "/esp/light" && val >= 1000){
        console.log("Ouvrir piscine");
      }
      else if (path == "/esp/light" && val < 1000){
        console.log("Fermé piscine");
      }
      if(path == "/esp/temp" && val >= 26){
        alert("Allez vous baignez !");
      }
    }



  }
};
</script>

<style>

h1{
  font-size: 40px;
}

#temp, #lum{
  background: #448aff;
  padding:20px;
  color:white;
  border-radius: 5px;
  margin-left: 5%;
  margin-right: 5%;
}

#valeur{
  font-weight: bold;
}
.contentPiscine{
  font-size:20px;
  float: left;
  width: 50%;
  padding-left: 5%;
}
.infoPiscine{
  font-size: 20px;
  float: right;
  width: 50%;
}

.voletPiscine{
  font-size: 25px;
  margin-top: 5%;
}
.ledPiscine{
  font-size: 25px;
  margin-top: 20%
}
#boutons{
  margin-top: 10%

}

</style>