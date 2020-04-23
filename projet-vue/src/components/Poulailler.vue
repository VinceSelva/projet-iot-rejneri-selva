<template>
  <div>
    <h1> Bienvenue sur le poulailler </h1>

      <div class="ledPoulailler">
        <p> Gestion du poulailler </p>
        <md-button class="md-primary md-raised" v-on:click="switchState('30:AE:A4:86:C3:20')">Ouvrir le poulailler</md-button>
        <br>
        <md-button class="md-primary md-raised" v-on:click="switchState('30:AE:A4:86:C3:20')">Fermer le poulailler</md-button>

      </div>

      <div class="mangerPoulailler">
        <p> Gestion nourriture </p>
        <md-button class="md-primary md-raised" v-on:click="switchState('30:AE:A4:86:C3:20')">Je sais pas quoi</md-button>
        <p> BlaBla </p>
        <p> BlaBla </p>
        <p> BlaBla </p>
        <p> BlaBla </p>
        <p> BlaBla </p>
      </div>

  </div>
</template>

<script>
export default {
  name: "Poulailler",
  data() {
    return {
      states: [],
      node_url: "http://51.83.77.127:3000",
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

  },
  props: {
    msg: String
  },
  mounted() {},

  methods: {
    start: function() {
      for (var i = 0; i < this.which_esps.length; i++) {
        this.getStates("/esp/light", [], this.which_esps[i]);
      }
      for (var j = 0; j < this.which_esps.length; j++) {
        this.getStates("/esp/temp", [], this.which_esps[j]);
      }
    },
    getStates(path_on_node, serie, wh) {
      this.node_url = "http://51.83.77.127:3000";

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
    }
  }
};
</script>

<style>

h1{
  font-size: 40px;
}

.ledPoulailler{
  font-size: 25px;
  float: left;
  margin-top: 300px;
  margin-left: 100px;
}
.mangerPoulailler{
  font-size: 25px;
  float: right;
  margin-top: 300px;
  margin-right: 100px;
}


.contentPoulailler{
  font-size: 18px;
  text-align: center;
}
</style>