<template>
  <div class="chatbox">
    <transition name="fade">
      <div class="chatboxSupport" v-show="isChatBotOpen">
        <div class="header">
          <div class="chatboxHeader">
            <div class="chatboxImageHeader">
              <img
                src="../Assets/chatbot-logo.png"
                alt="image"
                class="logo-size"
              />
            </div>
            <div class="chatbox__content--header">
              <h4 class="chatboxHeading">ASK Chatbot</h4>
            </div>
          </div>
          <div class="closeIcon">
            <Close @click="toggleChatbot()" fillColor="#ffffff" />
          </div>
        </div>
        <div class="chatboxMessages">
          <template v-for="(item, index) in messages" :key="item.message">
            <template v-if="item.name == 'Computer'">
              <template v-if="index == 0">
                <div class="messageContainer firstMessageContainer">
                  <div class="logo-bg">
                    <img
                      src="../Assets/chatbot-logo.png"
                      alt="image"
                      class="message-logo-size"
                    />
                  </div>
                  <div class="messageItem messageItemVisitor">
                    {{ item.message }}
                  </div>
                </div>
              </template>
              <template v-else-if="item.id == 0">
                <div class="messageContainer">
                  <div class="logo-bg">
                    <img
                      src="../Assets/chatbot-logo.png"
                      alt="image"
                      class="message-logo-size"
                    />
                  </div>
                  <div class="messageItem messageItemVisitor">
                    {{ item.message }}
                  </div>
                </div>
              </template>
              <template v-else>
                <div class="messageContainer">
                  <div class="logo-bg">
                    <img
                      src="../Assets/chatbot-logo.png"
                      alt="image"
                      class="message-logo-size"
                    />
                  </div>
                  <div>
                    <div class="messageItem messageItemVisitor">
                      {{ item.message }}
                    </div>
                    <div class="rating">
                      <div class="like" @click="toggleLike(item.id, true)">
                        <Thumbup
                          class="like"
                          :size="18"
                          :fillColor="!item.like ? '#000000' : '#ee4a62'"
                        />
                      </div>
                      <div class="dislike" @click="toggleLike(item.id, false)">
                        <Thumbdown
                          class="dislike"
                          :size="18"
                          :fillColor="!item.dislike ? '#000000' : '#ee4a62'"
                        />
                      </div>
                    </div>
                  </div>
                </div>
              </template>
            </template>
            <template v-else>
              <div class="prompt-message-container">
                <div class="messageItem messageItemOperator">
                  {{ item.message }}
                </div>
              </div>
            </template>
          </template>
        </div>
        <div class="chatboxFooter">
          <input
            type="text"
            placeholder="Ask anything..."
            v-model="data"
            @keyup.enter="getResponse()"
          />
          <button class="chatboxSendFooter sendButton">
            <img
              class="sendIcon"
              src="../Assets/send.svg"
              @click="getResponse()"
            />
          </button>
        </div>
      </div>
    </transition>

    <div class="chatboxButtonContainer">
      <button class="chatboxButton" @click="toggleChatbot()">
        <img src="../Assets/chatbot-logo.png" class="logo-size" />
      </button>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import Thumbup from "vue-material-design-icons/ThumbUp.vue";
import Thumbdown from "vue-material-design-icons/ThumbDown.vue";
import Close from "vue-material-design-icons/Close.vue";
export default {
  name: "Chatbot",
  data() {
    return {
      isChatBotOpen: false,
      data: "",
      messages: [
        {
          name: "Computer",
          id: 0,
          message:
            "Hey there. I am ASK, CIT's chatbot. How can I help you today? ",
          like: false,
          dislike: false,
        },
      ],
    };
  },
  components: {
    Thumbup,
    Thumbdown,
    Close,
  },
  methods: {
    toggleChatbot() {
      this.isChatBotOpen = !this.isChatBotOpen;
    },
    async getResponse() {
      if (String(this.data).length === 0) return;

      try {
        this.messages.push({ name: "User", id: 0, message: this.data });
        const userPrompt = this.data;
        this.data = "";
        const result = await axios.post(
          "http://34.125.178.209:3000/predict",
          { message: userPrompt },
          { withCredentials: true }
        );
        const message = {
          name: "Computer",
          id: result.data.id,
          message: result.data.answer,
          like: false,
          dislike: false,
        };
        this.messages.push(message);
      } catch (error) {
        const message = {
          name: "Computer",
          id: 0,
          message: "Something went wrong. Please try again later.",
        };
        this.messages.push(message);
      }
    },
    async toggleLike(transactionId, like) {
      const index = this.messages.findIndex(
        (message) => message.id == transactionId
      );
      const initialLike = this.messages[index].like;
      const initialDislike = this.messages[index].dislike;

      this.messages[index].like = like;
      this.messages[index].dislike = !like;

      try {
        await axios.patch(
          "http://34.125.178.209:3000/predict",
          {
            id: transactionId,
            like: like ? 0 : 1,
          },
          { withCredentials: true }
        );
      } catch (error) {
        console.log(error);
        this.messages[index].like = initialLike;
        this.messages[index].dislike = initialDislike;
      }
    },
  },
};
</script>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: "Nunito", sans-serif;
  font-weight: 400;
  background: #f1f1f1;
}

*,
html {
  --primaryBoxShadow: 0px 10px 15px rgba(0, 0, 0, 0.1);
  --secondaryBoxShadow: 0px -10px 15px rgba(0, 0, 0, 0.1);
  --primary: #581b98;
  --primaryColor: #ee4a62;
}

/* Transition of the chatbox */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
}

/* CHATBOX
=============== */
.chatbox {
  user-select: none;
  position: absolute;
  bottom: 30px;
  right: 30px;
}

/* CONTENT IS CLOSE */
.chatboxSupport {
  transform: translateY(-20px);
  display: flex;
  flex-direction: column;
  z-index: -123456;
  background: #f9f9f9;
  height: 500px;
  width: 350px;
  box-shadow: 0px 0px 15px rgba(0, 0, 0, 0.1);
  border-top-left-radius: 20px;
  border-top-right-radius: 20px;
}

/* BUTTON */
.chatboxButtonContainer {
  text-align: right;
}

.sendButton {
  padding: 5px;
  margin-left: 8px;
  background: var(--primaryColor);
  border-radius: 10px;
  border: none;
  outline: none;
  cursor: pointer;
}

.sendButton:hover {
  background-color: rgb(252, 34, 67);
}

.sendButton:active {
  background-color: var(--primaryColor);
}

.sendIcon {
  height: 20px;
  width: 22px;
  padding: 3px 1px 0px 2px;
}

/* HEADER */
.chatboxHeader {
  position: sticky;
  top: 0;
  background: orange;
  border-top-left-radius: 10px;
  background: var(--primaryColor);
  display: flex;
  flex-direction: row;
  align-items: center;
  padding: 5px 10px;
}

/* MESSAGES */
.chatboxMessages {
  margin-bottom: auto;
  display: flex;
  overflow-y: scroll;
  flex-direction: column;
  background-color: rgb(248 250 252);
  height: 100%;
  padding: 0 6px;
  overflow-x: hidden;
  overflow-wrap: break-word;
}
.firstMessageContainer {
  margin-top: 10px;
}

/* HEADER */
.header {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  background: var(--primaryColor);
  box-shadow: var(--primaryBoxShadow);
  border-top-left-radius: 10px;
  border-top-right-radius: 10px;
}

.closeIcon {
  cursor: pointer;
  display: none;
  padding: 10px;
}

.chatboxImageHeader {
  margin-right: 10px;
}

.chatboxHeading {
  font-size: 1.2rem;
  color: white;
}

.prompt-message-container {
  display: flex;
  align-items: end;
  justify-content: end;
  margin-top: 10px;
  margin-bottom: 20px;

  overflow: none;
}
.messageItemVisitor {
  margin-right: auto;
  border-radius: 0 10px 10px 10px;
}

.messageItem {
  margin-top: px;
  margin-left: 4px;
  padding: 8px 12px;
  box-shadow: rgba(251, 149, 149, 0.491) 0px 1px 2px 0px,
    rgba(60, 64, 67, 0.15) 0px 2px 6px 2px;
}

.messageItemOperator {
  border-radius: 10px 0 10px 10px;
  background: var(--primaryColor);
  color: white;
  margin-left: auto;
  max-width: 80%;
}

/* FOOTER */
.chatboxFooter {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  padding: 15px 10px;
  background: var(--secondaryGradient);
  box-shadow: var(--secondaryBoxShadow);
  border-bottom-right-radius: 10px;
  border-bottom-left-radius: 10px;
  position: sticky;
  bottom: 0;
}

.chatboxFooter input {
  width: 100%;
  border: none;
  padding: 10px 10px;
  border-radius: 10px;
  text-align: left;
}

.chatboxSendFooter {
  color: white;
}

.chatboxButtonContainer button,
.chatboxButtonContainer button:focus,
.chatboxButtonContainer button:visited {
  padding: 10px;
  background: white;
  border: none;
  outline: none;
  border-top-left-radius: 50px;
  border-top-right-radius: 50px;
  border-bottom-left-radius: 50px;
  box-shadow: 0px 10px 15px rgba(0, 0, 0, 0.1);
  cursor: pointer;
}

::-webkit-scrollbar {
  width: 5px;
}

/* Track */
::-webkit-scrollbar-track {
  background: #f1f1f1;
}

/* Handle */
::-webkit-scrollbar-thumb {
  background: rgb(199, 197, 197);
}

/* Handle on hover */
::-webkit-scrollbar-thumb:hover {
  background: rgb(237, 226, 226);
}

.rating {
  margin: 3px 10px;
}

.like,
.dislike {
  font-size: 1rem;
  display: inline-block;
  cursor: pointer;
}
.like {
  margin-right: 3px;
}

.dislike:hover,
.like:hover {
  color: #0042f9;
  transition: all 0.2s ease-in-out;
  transform: scale(1.1);
}

.dislike:active,
.like:active {
  color: #ee4a62;
}
.like:focus {
  color: #ee4a62;
}

.messageContainer {
  display: flex;
  align-items: start;
  justify-content: start;
  width: 80%;
}

.online {
  border-radius: 50%;
  height: 10px;
  width: 10px;
  background-color: rgb(48, 223, 48);
  position: absolute;
  bottom: 10px;
  right: 3px;
}

/* Logo */
.logo-size {
  width: 40px;
  height: 40px;
}

.message-logo-size {
  width: 27px;
  height: auto;
}

@media screen and (max-width: 600px) {
  .closeIcon {
    display: block;
  }

  .chatbox {
    margin: 0;
  }

  .chatboxSupport {
    height: 100vh;
    width: 100vw;
    transform: translateY(0);
    position: fixed;
    bottom: 0;
    left: 0;
    z-index: 1000;
  }

  .header {
    border-top-left-radius: 0;
    border-top-right-radius: 0;
  }
}
</style>
