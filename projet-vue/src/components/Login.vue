<template>
    <div id="login">
        <h1>Login</h1>
        <input type="text" name="username" v-model="input.username" placeholder="Nom d'utilisateur" />
        <input type="password" name="password" v-model="input.password" placeholder="Mot de passe" />
        <button type="button" v-on:click="login()">Login</button>
    </div>
</template>

<script>
    export default {
        name: 'Login',
        data() {
            return {
                input: {
                    username: "",
                    password: ""
                }
            }
        },
        methods: {
            login() {
                //Si l'utilisateur remplit les champs identifiants
                if(this.input.username != "" && this.input.password != "") {
                    //Si les identifiants sont bons
                    if(this.input.username == this.$parent.identifiants.username && this.input.password == this.$parent.identifiants.password) {
                        this.$emit("authenticated", true);
                        this.$router.replace({ name: "Accueil" });
                    } else {
                        console.log("Mot de passe incorrect !");
                        alert("Mot de passe incorrect !");
                    }
                } else {
                    alert("Veuillez renseigner vos identifiants");
                    console.log("Veuillez renseigner vos identifiants");
                }
            }
        }
    }
</script>

<style scoped>
    #login {
        width: 500px;
        border: 1px solid #CCCCCC;
        background-color: #FFFFFF;
        margin: auto;
        margin-top: 200px;
        padding: 20px;
    }
</style>