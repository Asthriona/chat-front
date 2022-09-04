<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
      <div class="d-flex align-center">
        <v-img
          @click="window.location.href = '/'"
          alt="Vuetify Name"
          class="shrink mt-1 hidden-sm-and-down"
          contain
          min-width="100"
          src="https://cdn.asthriona.com/i/2022/04/Asthriona%20logo%20black.png"
          width="300"
        />
      </div>

      <v-spacer></v-spacer>
      <v-btn to="/" text>Home</v-btn>
      <v-btn v-if="user" to="/chat" text>Chat </v-btn>
      <v-spacer></v-spacer>

      <v-btn
        text
      >
        <span class="mr-2">{{ user.username }}#{{ user.discriminator }}</span>
        <v-icon>{{ user.selectedBadge }}</v-icon>
      </v-btn>
    </v-app-bar>

    <v-main>
      <router-view :user="user" />
    </v-main>
  </v-app>
</template>

<script>
  import axios from 'axios';
  export default {
    name: 'App',
    data() {
      return {
        user: {}
      }
    },
    created() {
      axios.get(`${process.env.VUE_APP_GATE}/v2/login/whoami`, {
        headers: {
          Authorization: localStorage.getItem('token')
        }
      })
      .then(res => {
        this.user = res.data.user;
      })
      .catch(err => {
        console.log(err);
      })
    }
  }
  </script>
  
  <style>
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    color: #2c3e50;
  }
  nav {
    padding: 30px;
  }
  nav a {
    font-weight: bold;
    color: #2c3e50;
  }
  nav a.router-link-exact-active {
    color: #42b983;
  }
  .avatar {
    border-radius: 50%;
  }
  .text-center {
    text-align: center;
  }
  </style>
