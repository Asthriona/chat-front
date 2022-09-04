<template>
  <div class="chat-window">
    <div class="messages">
        <v-dialog
        v-model="dialog">
        <v-card>
            <v-card-text>
                <v-text-field
                v-model="editMessage.message"></v-text-field>
                <v-btn @click="editedMessageSubmit">Submit</v-btn>
            </v-card-text>
        </v-card>
    </v-dialog>
      <div class="message" v-for="message in messages" :key="message.msg">
        <!-- <div class="msg-avatar" v-if="message.user.avatar"><img :src="message.user.avatar" class="avatar" alt="" height="30px"></div> -->
        <div :class="`username ${message.user.isAdmin ? settings.admin.hiddenAdmin ? 'hidden-admin' : 'admin' : ''}`">{{ message.user.displayName }} <v-icon v-if="message.user.selectedBadge">{{ message.user.selectedBadge }}</v-icon>: </div>
        <div :class="`message-text ${message.deleted ? 'deleted' : ''} ${message.user.displayName == 'System' ? 'system-message' : ''}`">{{ message.msg }}</div>
        <div class="AdminAction" v-if="user.isAdmin">
        <v-btn icon><v-icon>mdi-shield-off-outline</v-icon></v-btn>
        <v-btn @click="messageActionDelete(message.index)" icon><v-icon>mdi-delete-forever</v-icon></v-btn>
    </div>
        <div class="message-action" v-if="message.user.id == user.id">
            <!-- <v-btn @click="messageActionEdit(message.index)" icon><v-icon>mdi-account-edit</v-icon></v-btn> -->
        </div>
      </div>
    </div>
    <form type="text" class="input-container" @submit="sendMessage">
      <input
        type="text"
        v-model="msg"
        placeholder="ここにメッセージを入力してください"
      />
      <button @click="sendMessage" :disabled="!msg">送る</button>
    </form>
  </div>
</template>

<script>
export default {
  name: "ChatRoom",
  props: ["messages", "settings", "user", "socket"],
  data() {
    return {
      msg: "",
      dialog: false,
      editMessage: {
          message: "",
          index: ""
      }
    };
  },
  methods: {
    sendMessage() {
      if (!this.msg) return;
    // check if message is empty or just spaces
    if (this.msg.trim() === "") return console.log("no.");
        this.$emit("sendMessage", this.msg);
      this.msg = "";
    },
    messageActionDelete(index) {
        if (confirm("Are you sure you want to delete this message?")) {
            console.log(index)
            this.socket.emit("messageDelete", index);

      }
    },
    messageActionEdit(index) {
            this.editMessage.message = this.messages[index].msg;
            this.editMessage.index = index;
            this.dialog = true;
    },
    editedMessageSubmit() {
        this.$emit("editedMessage", this.editMessage);
        this.dialog = false;
    }
  },
};
</script>

<style lang="scss" scoped>
.chat-window {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 1rem;
  background-color: #fff;
  box-shadow: 1px 1px 6px 0px rgba(0, 0, 0, 0.2);
  .messages {
    flex: 1;
    overflow: scroll;

    .message {
      display: flex;
      border-bottom: 1px solid #ccc;
      padding: 0.5rem 0;
      &:last-of-type {
        border-bottom: none;
      }
      .username {
        font-weight: bold;
        margin-right: 0.5rem;
      }
      .admin {
        color: red;
      }
      .system-message {
        color: red;
        font-weight: 900;
      }
      .message-text {
        flex: 1;

        .deleted {
          color: #ccc;
          text-decoration: line-through;
        }
      }
    }
  }
  .input-container {
    display: flex;

    input {
      flex: 1;
      height: 35px;
      font-size: 18px;
      box-sizing: border-box;
    }
    button {
      width: 75px;
      height: 35px;
      box-sizing: border-box;
    }
  }
}
</style>