<template>
  <div id="app">
      <router-link tag="li" class="col" class-active="active" to="/">Acceuil</router-link>
      <router-link tag="li" class="col" class-active="active" to="/graphiques">Voir les graphiques</router-link>
      <router-link tag="li" class="col" class-active="active" to="/piscine">Voir la piscine</router-link>
      <router-link tag="li" class="col" class-active="active" to="/poulailler">Voir le poulailler</router-link>
      <router-link tag="li" class="col" class-active="active" v-if="authenticated" to="/login" v-on:click.native="logout()" replace>Logout</router-link>
    
    <router-view @authenticated="setAuthenticated" />

  </div>
</template>


<script>
    export default {
        name: 'App',
        data() {
            return {
                authenticated: false,
                mockAccount: {
                    username: "test",
                    password: "admin"
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

 nav li:hover,
 nav li.router-link-active,
 nav li.router-link-exact-active {
   background-color: indianred;
   cursor: pointer;
   padding: 100px;
 }
</style>
