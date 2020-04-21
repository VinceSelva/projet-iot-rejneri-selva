<template>
  <div id="app">
    <router-link class="nav" to="/">Se connecter</router-link>
    <router-link class="nav" v-if="authenticated" to="/accueil">Acceuil</router-link>
    <router-link class="nav" v-if="authenticated" to="/graphiques">Voir les graphiques</router-link>
    <router-link class="nav" v-if="authenticated" to="/piscine">Voir la piscine</router-link>
    <router-link class="nav" v-if="authenticated" to="/poulailler">Voir le poulailler</router-link>
    <router-link class="nav" v-if="authenticated" to="/" v-on:click.native="logout()" replace>Logout</router-link>
    
    <router-view @authenticated="setAuthenticated" />

  </div>
</template>


<script>
    export default {
        name: 'App',
        data() {
            return {
                authenticated: false,
                identifiants: {
                    username: "test",
                    password: "test"
                }
            }
        },
        mounted() {
            if(!this.authenticated) {
                this.$router.replace({ name: "Login" });
            }
        },
        methods: {
            setAuthenticated(status) {
                this.authenticated = status;
            },
            logout() {
                this.authenticated = false;
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
  margin-top: 60px;
}

 nav {
    padding: 100px;
 }
</style>
