<!--Vue representant la page Poulailler-->
<template>
  <div>
    <h1> Bienvenue sur le poulailler</h1>
      <div id="Poulailler">
        <div class="contentPoulailler">
          <p>Voici votre page Poulailler.
            Ici vous avez accès à la température à l'intérieur du poulailler, ainsi que la luminosité ambiante.
          </p>
          <p>
            Votre poulailler se ferme automatiquement le soir quand la luminosité baisse, et il s'ouvre le matin au levé du soleil. Au moins les poules ne restent pas enfermé toutes la journée.
            Le deuxième bouton permet d'allumer une lampe si vous voulez allez voir le poulailler la nuit.
          </p>
          <p>
            De plus, vous pouvez depuis cette page nourrir les poules à n'importe quel moment et n'importe où.
            Vous avez aussi la possibilité de démarrer la ponte automatique des oeufs, ce qui forcera les poules à pondre toutes les 30 minutes.
          </p>

          <hr>
             <p> Gestion du poulailler </p>
    <label style="margin-left: 300px;" class="switch">
      <input class="switch-input" type="checkbox" v-on:click="switchState('30:AE:A4:86:C3:20')"/>
      <span class="switch-label" data-on="On" data-off="Off"></span> 
      <span class="switch-handle"></span> 
    </label>
        <!--<md-button class="md-primary md-raised" v-on:click="ouverturePoulailler()">Ouvrir le poulailler</md-button>
        <br>
        <md-button class="md-primary md-raised" v-on:click="fermeturePoulailler()">Fermer le poulailler</md-button>-->
        <p>Etat du poulailler : {{etatPoulailler}}</p>
        <img id="poulaillerOuvert" height="500px" width="500px" src="../assets/poulailler-jour.jpg" alt="poulailler-jour" style="display: none;">
         <img id="poulaillerFermee" height="500px" width="500px" src="../assets/poulailler-nuit.jpg" alt="poulailler-jour" style="display: inline;">

        </div>
        <div class="infoPoulailler">


        <p>Devenez membre en adhérant !</p>
		<label for="macPoulailler">Votre adresse MAC : </label>
		<input type="text" id="macPoulailler" name="macPoulailler" v-model="selected">

        <p id="temp">Température : <span id="valeur">{{lastTemp}}</span> °C</p>
        <p id="lum">Luminosité : <span id="valeur">{{lastLuminosite}}</span> Lum</p>

        <div><hr></div>

        <p> Gestion de la nourriture  </p>
        <md-button class="md-primary md-raised" v-on:click="debutNourrir()">Nourrir les poules automatiquement</md-button>
        <p>Les poules ont été nourrit {{ nourrir }} fois.</p>

        <p> Gestion des œufs  </p>
        <md-button class="md-primary md-raised" v-on:click="debutPonte()">Debuter la ponte automatique</md-button>
        <p>Nombre d'oeufs pondus : {{ oeufs }}</p>



        </div>
      </div>
     </div>
</template>

<script>

export default {
  name: "Poulailler",
  data() {
    return {
      /* eslint-disable no-mixed-spaces-and-tabs */
          selected: '30:AE:A4:86:C3:20',

     	ponteEnCours: false,
      interval: null,
      states: [],
      lastTemp: null,
      lastLuminosite: null,
      etatPoulailler: "Fermé",
      oeufs: 0,
      nourrir: 0,
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
 mounted() {
    if (localStorage.oeufs) {
      this.oeufs = localStorage.oeufs;
    }
    if (localStorage.nourrir) {
      this.nourrir = localStorage.nourrir;
    }
  },
  watch: {
    oeufs(newName) {
      localStorage.oeufs = newName;
    },
    nourrir(newName) {
      localStorage.nourrir = newName;
    }
  }, 
  methods: {
    start: function() {
      for (var i = 0; i < this.which_esps.length; i++) {
        this.get_states("/esp/light", [], this.which_esps[i]);
      }
      for (var j = 0; j < this.which_esps.length; j++) {
        this.get_states("/esp/temp", [], this.which_esps[j]);
      }
      /*for (var k = 0; k < 20; j++) {
        nourrir++;
      }*/
    },

    debutPonte() {
      if (this.ponteEnCours) {
        clearInterval(this.interval);
      } else {
        this.interval = setInterval(this.ponteOk, 30000);
      }
      this.ponteEnCours = !this.ponteEnCours
    },
    ponteOk() {
      this.oeufs = parseInt(this.oeufs) + 1;
    },

    debutNourrir() {
      if (this.nourrirEnCours) {
        clearInterval(this.interval);
      } else {
        this.interval = setInterval(this.nourrirOK, 30000);
      }
      this.nourrirEnCours = !this.nourrirEnCours
    },
    nourrirOK() {
      this.nourrir = parseInt(this.nourrir) + 1;
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
      var esp = "30:AE:A4:86:C3:20";
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
       //console.log("Last " + path + " : " + val);
    },

    /*get_states(path_on_node, serie, wh) {

      this.node_url = "http://51.83.77.127:3000";

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
          if(liste !== undefined){
          	this.lastValue(path_on_node,(liste[(liste.length-1)][1]));
          }

        });
    },*/
    get_states(path_on_node, serie, wh) {

      this.node_url = "http://51.210.15.67:3000";

      var liste = []
      //wh = "30:AE:A4:86:C3:20";
      wh = this.selected;
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
          //console.log("Liste"+liste);
          if(liste !== undefined)
            this.lastValue(path_on_node,(liste[(liste.length - 1)][1]));

        });
    },
    ouverturePoulailler(){
        this.etatPoulailler = "Ouvert";
        document.getElementById("poulaillerOuvert").style.display = 'inline';
        document.getElementById("poulaillerFermee").style.display = 'none';
    },
    fermeturePoulailler(){
        this.etatPoulailler = "Fermé";
        document.getElementById("poulaillerFermee").style.display = 'inline';
        document.getElementById("poulaillerOuvert").style.display = 'none';
    },
    test(path, val){
      //console.//log("test " + path + " : " + val);

      if(path == "/esp/light" && val >= 1000){
        //this.etatVolet = "Ouvert";
        //console.log("Ouvrir piscine");
        this.ouverturePoulailler();
      }
      else if (path == "/esp/light" && val < 1000){
        //this.etatVolet = "Fermé";
        //console.log("Fermé piscine");
        this.fermeturePoulailler();
      }
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


.contentPoulailler{
  font-size: 18px;
  text-align: center;
}
</style>