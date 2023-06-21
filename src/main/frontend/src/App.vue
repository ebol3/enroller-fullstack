<template>
  <div id="app">
    <h1>Witaj w systemie do zapisów na zajęcia</h1>

    <div v-if="authenticatedUsername">
      <UserPanel :username="authenticatedUsername" @logout="logMeOut()"></UserPanel>
      <MeetingsPage :username="authenticatedUsername"></MeetingsPage>
    </div>

    <div v-else>
      <button :class="signingUp ? 'button-outline' : ''" @click="signingUp=false">Logowanie</button>
      <button :class="!signingUp ? 'button-outline' : ''" @click="signingUp=true">Rejestracja</button>

      <div v-if="message" :class="created ? 'alert' : 'alert2'">{{message}}</div>

      <LoginForm v-if="!signingUp" @login="(user) => logMeIn(user)"></LoginForm>
      <LoginForm v-else @login="(user) => register(user)" button-label="Załóż konto"></LoginForm>
    </div>
  </div>
</template>

<script>
import "milligram";
import LoginForm from "./LoginForm";
import UserPanel from "./UserPanel";
import MeetingsPage from "./meetings/MeetingsPage";
import axios from "axios";


export default {
  components: {LoginForm, MeetingsPage, UserPanel},
  data() {
    return {
      message:"",
      created: false,
      signingUp: false,
      authenticatedUsername: '',
      meetings: [],
    }
  },
  methods: {
    logMeIn(user) {
      axios.post('/api/tokens', user)
          .then(response => {
            this.authenticatedUsername = user.login;
            const token = response.data.token;
            axios.defaults.headers.common["Authorization"] = 'Bearer ' + token;
            axios.get('/api/meetings')
                .then(response => {
                  this.meetings = response.data;
          })
                .catch(response => {
                  this.created = true;
                  this.message = ("Nie pobrano listy spotkań")
                })
            this.created = false;
            this.message = ("Logowanie udane.")
          })
          .catch(response => {
            this.created = true;
            this.message = ("Logowanie nieudane.")
          })
    },
    logMeOut() {
      this.authenticatedUsername = '';
      delete axios.defaults.headers.common.Authorization;
    },
    register(user) {
      axios.post('/api/participants', user)
          .then(response => {
            this.message = "Udało się utworzyć konto."
            this.created = true
            // udało się
          })
          .catch(response => {
            this.message = "Nie udało się utworzyć konta!."
            this.created = false
            // nie udało sie
          });
    }
  }
}
</script>

<style>
#app {
  max-width: 1000px;
  margin: 0 auto;
}
.alert{
  display: flex;
  text-align: center;
  color: green;
  padding: 20px;
  margin: 40px;
  box-shadow: rgba(0, 0, 0, 0.25) 0px 54px 55px, rgba(0, 0, 0, 0.12) 0px -12px 30px, rgba(0, 0, 0, 0.12) 0px 4px 6px, rgba(0, 0, 0, 0.17) 0px 12px 13px, rgba(0, 0, 0, 0.09) 0px -3px 5px;
}
.alert2{
  display: flex;
  text-align: center;
  color: red;
  padding: 20px;
  margin: 40px;
  /*box-shadow: rgba(0, 0, 0, 0.25) 0px 54px 55px, rgba(0, 0, 0, 0.12) 0px -12px 30px, rgba(0, 0, 0, 0.12) 0px 4px 6px, rgba(0, 0, 0, 0.17) 0px 12px 13px, rgba(0, 0, 0, 0.09) 0px -3px 5px;*/
}
</style>
