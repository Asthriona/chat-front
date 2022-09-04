<template>
    <div class="home text-center">
      <h1>cum.</h1>
      <div class="login" v-if="!user.username">
        <p>ログインしてください</p>
      <h2 class="error">{{error.message}}</h2>
      <form>
      <input v-model="email" type="text" placeholder="email">
      <input v-model="password" type="password" placeholder="password">
      <button type="submit" @click="Login">ログイン</button>
      </form>
      </div>
      <div class="user" v-else>
        <img class="avatar" :src="user.avatar" :alt="`${user.displayName}'s Avatar'`" height="100px">
        <p>ようこそ{{user.displayName}}さん</p>
      </div>
    </div>
</template>

<script>
import axios from 'axios';

export default {
  name: "homeView",
  props: ["user"],
  data() {
    return { 
      email: '',
      password: '',
      error: {
        message: ''
      }
    }
  },
  methods: {
    Login(e) {
      e.preventDefault();
      axios.post(`${process.env.VUE_APP_GATE}/v2/login/login`, {
        email: this.email,
        password: this.password
      })
      .then(res => {
        localStorage.setItem('token', res.data.token)
        this.error = {
          message: 'You are now logged in, redirecting...'
        }
        return setTimeout(() => {
          window.location.href = '/';
        }, 1000)
      })
      .catch(err => {
        this.error = {
          message: err.response.data.message
        }
      })
    }
  }
}
</script>