<template>
  <div>
    <h1> Bienvenue sur la piscine </h1>
      <div class="contentPiscine">
        <p>Le Lorem Ipsum est simplement du faux texte employé dans la composition et la mise en page avant impression. 
        Le Lorem Ipsum est le faux texte standard de l'imprimerie depuis les années 1500, quand un imprimeur anonyme assembla ensemble des 
        morceaux de texte pour réaliser un livre spécimen de polices de texte. Il n'a pas fait que survivre cinq siècles, mais s'est aussi 
        adapté à la bureautique informatique, sans que son contenu n'en soit modifié. Il a été popularisé dans les années 1960 grâce à la 
        vente de feuilles Letraset contenant des passages du Lorem Ipsum, et, plus récemment, par son inclusion dans des applications de 
        mise en page de texte, comme Aldus PageMaker.
        </p>
        <p>
        Le Lorem Ipsum est simplement du faux texte employé dans la composition et la mise en page avant impression. 
        Le Lorem Ipsum est le faux texte standard de l'imprimerie depuis les années 1500, quand un imprimeur anonyme assembla ensemble des 
        morceaux de texte pour réaliser un livre spécimen de polices de texte. Il n'a pas fait que survivre cinq siècles, mais s'est aussi 
        adapté à la bureautique informatique, sans que son contenu n'en soit modifié. Il a été popularisé dans les années 1960 grâce à la 
        vente de feuilles Letraset contenant des passages du Lorem Ipsum, et, plus récemment, par son inclusion dans des applications de 
        mise en page de texte, comme Aldus PageMaker.</p>
      </div>

      <div class="ledPiscine">
        <p> Led </p>
        <md-button class="md-primary md-raised" v-on:click="switchState('30:AE:A4:86:C3:20')">Allumer la led</md-button>
        <br>
        <md-button class="md-primary md-raised" v-on:click="switchState('30:AE:A4:86:C3:20')">Eteindre la led</md-button>
      </div>

      <div class="tempPiscine">
        <p> Gestion de la piscine </p>

        <p id="">Température de l'eau : {{lastTemp}} °C</p>
        <p id="">Luminosité : {{lastLuminosite}} Lum</p>

      </div>

      <div class="voletPiscine">
        <p> Gestion du volet </p>
        <md-button class="md-primary md-raised" v-on:click="switchState('30:AE:A4:86:C3:20')">Ouvrir la piscine</md-button>
        <br>
        <md-button class="md-primary md-raised" v-on:click="switchState('30:AE:A4:86:C3:20')">Fermer la piscine</md-button>
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

.ledPiscine{
  font-size: 25px;
  float: left;
  margin-top: 300px;
  margin-left: 100px;
}

.tempPiscine{
  font-size: 25px;
  float: right;
  margin-top: 300px;
  margin-right: 100px;
}

.voletPiscine{
  font-size: 25px;
  margin-top: 300px;
  margin-right: 100px;
}


.contentPiscine{
  font-size: 18px;
  text-align: center;
}
</style>