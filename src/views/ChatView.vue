<template>
  <v-container>
    <v-row>
      <div class="alert" v-if="userAlert">
        <v-alert
          :icon="userAlert.icon"
          transition="fade-transition"
          :type="userAlert.type"
          dismissible
          >{{ userAlert.message }}</v-alert
        >
      </div>
      <v-col cols="8">
        <div class="Chat">
          <div class="ban-message" v-if="isBanned.status">
            <h1>You have been banned from the chat.</h1>
            <p>Reason: {{ isBanned.reason }}</p>
            <p>Time: {{ isBanned.displayTime }}</p>
            <p>
              Staff: {{ isBanned.staff.displayName }}#{{
                isBanned.staff.discriminator
              }}
            </p>
          </div>
          <div class="chatBot">
            <ChatRoom
              v-if="isOffline == false"
              :class="isBanned.status == true ? 'Banned' : ''"
              v-on:sendMessage="this.sendMessage"
              :socket="socket"
              :messages="messages"
              :settings="settings"
              :user="user"
            />
          </div>
          <div class="offline" v-if="isOffline == true">
            <h1>The chatbox is currently offline.</h1>
          </div>
        </div>
      </v-col>
      <v-col cols="4">
        <div class="header">
          <h1>Asthriona のチャット ルーム</h1>
          <p class="username">あなたの名前: {{ user.displayName }}</p>
          <p class="online">現在オンラインのユーザー: {{ users.length }}</p>
          <ul>
            <li v-for="chatUser in users" :key="chatUser">{{ chatUser }}</li>
          </ul>
        </div>
        <v-card>
          <v-card-title>Settings</v-card-title>
          <v-card-text>
            <!-- mute notification sound -->
            <v-switch
              v-model="settings.muteNotificationSound"
              label="Mute Notification Sound"
              color="primary"
            />
          </v-card-text>
          <v-card-text v-if="user.isAdmin">
            <v-switch
              v-model="settings.admin.hiddenAdmin"
              label="Hide Admin"
              color="primary"
            />
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>
  
  <script>
// @ is an alias to /src

import io from "socket.io-client";
import ChatRoom from "../components/ChatRoom.vue";
export default {
  name: "ChatView",
  components: {
    ChatRoom,
  },
  props: ["user"],
  data() {
    return {
      socket: io(process.env.VUE_APP_WS),
      isJoined: "",
      messages: [],
      users: [],
      isOffline: false,
      userAlert: "",
      unread: 0,
      settings: {
        muteNotificationSound: false,
        admin: {
          hiddenAdmin: false,
        }
      },
      isBanned: {
        status: false,
        displayTime: "",
        reason: "",
        time: "",
        staff: {},
      },
    };
  },
  methods: {
    // join server
    joinServer() {
      if (this.isJoined) return;
      this.socket.on("loggedIn", (data) => {
        this.messages = data.messages;
        this.users = data.users;
        this.isOffline = false;
        this.isJoined = true;
        this.socket.emit("newUser", this.user);
      });
      this.listen();
    },
    // Listen for events
    listen() {
      // get user online
      this.socket.on("userOnline", (user) => {
        this.users.push(user);
        this.userAlert = {
          message: `${user} has joined the chat`,
          type: "success",
          icon: "mdi-account-plus",
        };
        setTimeout(() => {
          this.userAlert = "";
        }, 3000);
      });
      // get leaving user
      this.socket.on("userLeft", (user) => {
        this.userAlert = {
          message: `${user} has left the chat`,
          type: "error",
          icon: "mdi-account-minus",
        };
        this.users.splice(this.users.indexOf(user), 1);
        setTimeout(() => {
          this.userAlert = "";
        }, 3000);
      });
      // get new messages
      this.socket.on("message", (msg) => {
        this.messages.push(msg);
        // notification sound & title badge
        if (!document.hasFocus()) {
          if (msg.user.displayName == this.user.displayName) return;
          if (msg.user.displayName == "システム") return;
          // change window title
          this.unread++;
          document.title = `(${this.unread}) Asthriona のチャット ルーム`;
          if (this.settings.muteNotificationSound == false) {
            let audio = new Audio(
              "https://cdn.asthriona.com/i/2022/09/_pn_220904_0404AM46402.mp3"
            );
            audio.play();
          }
          // reset title once window is focused
          window.onfocus = () => {
            document.title = "Asthriona のチャット ルーム";
            this.unread = 0;
          };
        }
      });
      // Admin disconnect
      // This is an event for admin commands only, it will show the user as disconnected and remove them from the user list
      this.socket.on("fakeDisconnect", (data) => {
        this.userAlert = {
          message: `${data} has left the chat`,
          type: "error",
          icon: "mdi-account-minus",
        };
        this.users.splice(this.users.indexOf(data), 1);
        setTimeout(() => {
          this.userAlert = "";
        }, 3000);
      });
      // clear chat
      this.socket.on("clear", () => {
        this.messages = [];
      });
      // delete message
      this.socket.on("deletedMessage", (data) => {
        console.log(data);
        this.messages.splice(this.messages.indexOf(data), 1);
      });
      // whern a user is banned
      this.socket.on("BannedUser", (user) => {
        this.userAlert = {
          message: `${user} has been banned from the chat`,
          type: "error",
          icon: "mdi-shield-off-outline",
        };
        console.log(user);
        if (user.userId == this.user.id) {
          this.isBanned.status = true;
          this.isBanned.reason = user.reason;
          this.isBanned.time = user.time;
          this.isBanned.staff = user.staff;
          this.banCountDown(this.isBanned.time);
        }
        setTimeout(() => {
          this.userAlert = "";
        }, 3000);
      });
      // when a message got deleted.
      this.socket.on("messageDelete", (data) => {
        // remove message from array
          this.messages.splice(data.index, 1);
      });
    },
    // Send message on event
    sendMessage(msg) {
      this.socket.emit("message", msg);
    },
    // Ban user
    banCountDown(time) {
      time -= 1000;
      let countDown = setInterval(() => {
        if (time <= 0) {
          clearInterval(countDown);
          this.isBanned.status = false;
          this.isBanned.reason = "";
          this.isBanned.time = 0;
          this.isBanned.staff = {
            displayName: "",
            discriminator: "",
          };
        }
        // convert time from milliseconds to minutes and seconds
        let minutes = Math.floor(time / 60000);
        let seconds = ((time % 60000) / 1000).toFixed(0);
        this.isBanned.displayTime = `${minutes}:${
          seconds < 10 ? "0" : ""
        }${seconds}`;
        time -= 1000;
      }, 1000);
    },
    // on editedMessage
    editedMessage(msg) {
      this.socket.emit("editedMessage", msg);
    },
    
  },
  mounted() {
    document.title = "Asthriona のチャット ルーム";
    this.joinServer();
  },
  beforeDestroy() {
    this.socket.disconnect();
  },
};
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
.alert {
  position: absolute;
  width: 100%;
  left: 80%;
  z-index: 100;
}
.Banned {
  // blur
  filter: blur(5px);
}
.ban-message {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
  color: #000;
  font-size: 2rem;
  font-weight: 700;
  z-index: 100;
}
</style>