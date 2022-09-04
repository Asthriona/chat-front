<template>
  <div class="chat-window">
    <div class="messages">
      <div class="message" v-for="message in messages" :key="message.msg">
        <div class="msg-avatar"><img :src="message.user.avatar" class="avatar" alt="" height="30px"></div>
        <div :class="`username ${message.user.isAdmin ? 'admin' : ''}`">{{ message.user.displayName }}:</div>
        <div class="message-text" v-html="message.msg"></div>
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
  props: ["messages"],
  data() {
    return {
      msg: "",
    };
  },
  methods: {
    sendMessage() {
      if (!this.msg) return;
    // check if message is empty or just spaces
    if (this.msg.trim() === "") return this.msg.push({ username: "System", msg: "メッセージを入力してください"});
      this.$emit("sendMessage", this.msg);
      this.msg = "";
    },
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
      .message-text {
        flex: 1;
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