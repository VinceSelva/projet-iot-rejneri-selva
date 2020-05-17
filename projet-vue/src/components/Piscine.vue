<!--Vue representant la page Piscine-->
<template>
  <div>
    <!--eslint-disable no-mixed-spaces-and-tabs-->
	  <h1> Bienvenue sur votre piscine </h1>


	  <div id="Piscine">
    	<div class="contentPiscine">
      	<p>Voici votre page Piscine.
        	Ici vous avez accès à la température de l'eau, ainsi que la luminosité ambiante.
      	</p>
      	
      	<p>
          Grâce aux boutons ci-dessous, vous pouvez fermer le volet de manière manuelle.
          Le deuxième bouton permet d'allumer les lumières présentes aux alentours de votre piscine.
        </p>

        <hr>

			<p> Lumières ambiante </p>
    		<!--<md-button class="md-primary md-raised" v-on:click="switchState('30:AE:A4:86:CA:7C')">ON / OFF</md-button>
    		<p> Etat de la lumière : {{etatLumiere}} </p>-->
    		<label style="margin-left: 40%;" class="switch">
				  <input class="switch-input" type="checkbox" v-on:click="switchState('30:AE:A4:86:CA:7C')"/>
				  <span class="switch-label" data-on="On" data-off="Off"></span> 
				  <span class="switch-handle"></span> 
			  </label>

    	</div>
			<div class="infoPiscine">

        <p>Devenez membre en adhérant !</p>
         <label for="macPiscine">Votre adresse MAC : </label>
           <input type="text" id="macPiscine" name="macPiscine" v-model="selected">

				<p id="temp">Température de l'eau : <span id="valeur">{{lastTemp}}</span> °C</p>
				<p id="lum">Luminosité : <span id="valeur">{{lastLuminosite}}</span> Lum</p>
				<hr>
			</div>

	    </div>
	  	<div class="voletPiscine">
	    	<p> Gestion du volet </p>
	    	<md-button class="md-primary md-raised" v-on:click="ouvertureVolet()">Ouvrir la piscine</md-button>
	    	<md-button class="md-primary md-raised" v-on:click="fermetureVolet()">Fermer la piscine</md-button>
	    	<p> Etat du volet : {{etatVolet}} </p>
        <img id="piscineOuverte" height="500px" width="500px" src="../assets/piscine-jour.jpg" alt="piscine-jour" style="display: none;">
        <img id="piscineFermee" height="500px" width="500px" src="../assets/piscine-nuit.jpg" alt="piscine-jour" style="display: inline;">
	  	</div>
    </div> 


</template>


<script>



export default {
  name: "Piscine",
  data() {
    return {
      states: [],
      selected: '30:AE:A4:86:CA:7C',
      lastTemp: null,
      lastLuminosite: null,
      etatLumiere: "Eteinte",
      etatVolet: "Fermé",
      node_url: "http://51.210.15.67:3000",
      items: [],
      listeData: [],
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
       // this.process_esp();
    },


  props: {
    msg: String
  },
  mounted() {
    this.process_esp();
  },
  destroyed(){
    console.log("Sortie piscine");
  },

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
       //this.test(path, val);
       console.log("Last " + path + " : " + val);
    },

    get_states(path_on_node, serie, wh) {

      this.node_url = "http://51.210.15.67:3000";

      var liste = []
      //wh = "30:AE:A4:86:CA:7C";
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
    //test(path, val){
      //console.//log("test " + path + " : " + val);

      /*if(path == "/esp/light" && val >= 1000){
        this.etatVolet = "Ouvert";
        console.log("Ouvrir piscine");
      }
      else if (path == "/esp/light" && val < 1000){
        this.etatVolet = "Fermé";
        console.log("Fermé piscine");
      }
      if(path == "/esp/temp" && val >= 26){
        alert("Allez vous baignez !");
      }*/

    //},
    ouvertureVolet(){
        this.etatVolet = "Ouvert";
        document.getElementById("piscineOuverte").style.display = 'inline';
        document.getElementById("piscineFermee").style.display = 'none';
    },
    fermetureVolet(){
        this.etatVolet = "Fermé";
        document.getElementById("piscineFermee").style.display = 'inline';
        document.getElementById("piscineOuverte").style.display = 'none';
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
  float: left;
}

/*----------------------------*/
.switch {
	position: relative;
	display: block;
	vertical-align: top;
	width: 100px;
	height: 30px;
	padding: 3px;
	margin: 0 10px 10px 0;
	background: linear-gradient(to bottom, #eeeeee, #FFFFFF 25px);
	background-image: -webkit-linear-gradient(top, #eeeeee, #FFFFFF 25px);
	border-radius: 18px;
	box-shadow: inset 0 -1px white, inset 0 1px 1px rgba(0, 0, 0, 0.05);
	cursor: pointer;
	box-sizing:content-box;
}
.switch-input {
	position: absolute;
	top: 0;
	left: 0;
	opacity: 0;
	box-sizing:content-box;
}
.switch-label {
	position: relative;
	display: block;
	height: inherit;
	font-size: 10px;
	text-transform: uppercase;
	background: #eceeef;
	border-radius: inherit;
	box-shadow: inset 0 1px 2px rgba(0, 0, 0, 0.12), inset 0 0 2px rgba(0, 0, 0, 0.15);
	box-sizing:content-box;
}
.switch-label:before, .switch-label:after {
	position: absolute;
	top: 50%;
	margin-top: -.5em;
	line-height: 1;
	-webkit-transition: inherit;
	-moz-transition: inherit;
	-o-transition: inherit;
	transition: inherit;
	box-sizing:content-box;
}
.switch-label:before {
	content: attr(data-off);
	right: 11px;
	color: #aaaaaa;
	text-shadow: 0 1px rgba(255, 255, 255, 0.5);
}
.switch-label:after {
	content: attr(data-on);
	left: 11px;
	color: #FFFFFF;
	text-shadow: 0 1px rgba(0, 0, 0, 0.2);
	opacity: 0;
}
.switch-input:checked ~ .switch-label {
	background: #E1B42B;
	box-shadow: inset 0 1px 2px rgba(0, 0, 0, 0.15), inset 0 0 3px rgba(0, 0, 0, 0.2);
}
.switch-input:checked ~ .switch-label:before {
	opacity: 0;
}
.switch-input:checked ~ .switch-label:after {
	opacity: 1;
}
.switch-handle {
	position: absolute;
	top: 4px;
	left: 4px;
	width: 28px;
	height: 28px;
	background: linear-gradient(to bottom, #FFFFFF 40%, #f0f0f0);
	background-image: -webkit-linear-gradient(top, #FFFFFF 40%, #f0f0f0);
	border-radius: 100%;
	box-shadow: 1px 1px 5px rgba(0, 0, 0, 0.2);
}
.switch-handle:before {
	content: "";
	position: absolute;
	top: 50%;
	left: 50%;
	margin: -6px 0 0 -6px;
	width: 12px;
	height: 12px;
	background: linear-gradient(to bottom, #eeeeee, #FFFFFF);
	background-image: -webkit-linear-gradient(top, #eeeeee, #FFFFFF);
	border-radius: 6px;
	box-shadow: inset 0 1px rgba(0, 0, 0, 0.02);
}
.switch-input:checked ~ .switch-handle {
	left: 74px;
	box-shadow: -1px 1px 5px rgba(0, 0, 0, 0.2);
}
 
/* Transition
========================== */
.switch-label, .switch-handle {
	transition: All 0.3s ease;
	-webkit-transition: All 0.3s ease;
	-moz-transition: All 0.3s ease;
	-o-transition: All 0.3s ease;
}
</style>