<template>
    <div class="Chat">
      <div class="header">
        <h1>Chatroom</h1>
        <p class="username">Username: {{ user.displayName }}</p>
        <p class="online">Online: {{ users.length }}</p>
        <!-- <ul>
          <li v-for="chatUser in users" :key="chatUser.displayName">{{ chatUser }}</li>
        </ul> -->
        <ChatRoom v-on:sendMessage="this.sendMessage" :messages="messages" />
      </div>
    </div>
  </template>
  
  <script>
  // @ is an alias to /src
  
  import io from 'socket.io-client';
  import ChatRoom from "../components/ChatRoom.vue";
  export default {
    name: 'ChatView',
    components: {
      ChatRoom
    },
    props: ["user"],
    data() {
      return {
        socket: io('http://localhost:2000'),
        messages: [],
        users: []
      }
    },
    methods: {
      // random String generator
      randomString(length) {
        let chars = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
        let result = '';
        let withNumbers;
        Math.floor(Math.random() * 2) === 0 ? (withNumbers = false) : (withNumbers = true)
        if (withNumbers) chars += '0123456789';
        for (let i = length; i > 0; --i) result += chars[Math.floor(Math.random() * chars.length)];
        return result;
      },
      joinServer() {
        this.socket.on("loggedIn", data => {
          this.messages = data.messages;
          this.users = data.users;
          this.socket.emit('newUser', this.user);
        });
        this.listen();
      },
      listen() {
        this.socket.on("userOnline", user => {
          console.log(user)
          this.users.push(user);
        })
        this.socket.on("userLeft", user => {
          this.users.splice(this.users.indexOf(user), 1);
        });
        this.socket.on("message", msg => {
            this.messages.push(msg);
        });
        this.socket.on("clear", () => {
          this.messages = [];
        });
        this.socket.on("deletedMessage", (data) => {
          console.log(data)
          this.messages.splice(this.messages.indexOf(data), 1);
        })
      },
      sendMessage(msg) {
        this.socket.emit("message", msg);
      }
    },
    mounted() {
      this.joinServer();
    }
  }
  </script>
  
  <style lang="scss" scoped>
  body {
    font-family: "Avenir", Helvetica, Arial, sans-serif;
    color: #2c3e50;
    margin: 0;
    padding: 0;
  }
  #app {
    display: flex;
    flex-direction: column;
    height: 100vh;
    width: 100%;
    max-width: 768px;
    margin: 0 auto;
  }
  </style>