<template>
  <div id="app">
    <nav>
      <router-link to="/">Home</router-link> |
      <router-link to="/chat" v-if="user">Chat Room</router-link>
      <router-link to="/" v-else>Chat Room</router-link>
      
    </nav>
    <router-view :user="user" />
  </div>
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
    axios.get(`http://localhost:3000/api/v2/login/whoami`, {
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
