<template>
  <div id="app">
  <div id="menu">
    <router-link to="/">Accueil</router-link>
    <router-link to="/graphiques">Voir les graphiques</router-link>
    <router-link to="/piscine">Voir la piscine</router-link>
    <router-link to="/poulailler">Voir le poulailler</router-link>
    </div>
    <router-view></router-view>
         </div>
</template>

<script>


export default {
  name: "App",
  components: {
  }
 
}




</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

#menu{
  
}
</style>
