<template>
  <div id="app">
    <md-toolbar v-if="authenticated">
      <router-link class="md-title nav" v-if="authenticated" to="/accueil">Accueil</router-link>
      <router-link class="md-title nav" v-if="authenticated" to="/piscine">Voir la piscine</router-link>
      <router-link class="md-title nav" v-if="authenticated" to="/poulailler">Voir le poulailler</router-link>
      <router-link class="md-title nav" v-if="authenticated" to="/graphiques">Voir les graphiques</router-link>
      <h3 class="nav-right2" v-if="authenticated">Bonjour {{ identifiants.username }} !</h3>
      <router-link class="md-title nav-right1" v-if="authenticated" to="/" v-on:click.native="logout()" replace>Se déconnecter</router-link>
    </md-toolbar>

    <router-view @authenticated="setAuthenticated" />
  </div>
  
</template>


<script>
    export default {
        name: 'App',
        data() {
            return {
                authenticated: false,
                authenticatedUser: false,
                identifiants: {
                    username: "admin",
                    password: "admin",
                    usernameUser: "user",
                    passwordUser: "user"
                }
            }
        },
        mounted() {
            if(!this.authenticated) {
              this.$router.push("/")
            }
            if(!this.authenticatedUser) {
              this.$router.push("/")
            }
        },
        methods: {
            setAuthenticated(status) {
                this.authenticated = status;
            },
            setAuthenticatedUser(status) {
                this.authenticatedUser = status;
            },
            logout() {
                this.authenticated = false;
                this.authenticatedUser = false;
            }
    
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
}

h1{
  font-size: 40px;
  padding-top:50px;
}

.md-toolbar{
  padding: 10px !important;
}

.nav{
  padding:20px;
}

.nav:hover{
  text-decoration: none !important;
  background-color: #cccccc;
  color: white !important;
  border-radius: 10px;
  cursor: pointer;
}

.nav-right1 {
    margin-left: 20% !important;
 }
 .nav-right2 {
    margin-left: 20% !important;
    cursor: pointer;
 }


</style>