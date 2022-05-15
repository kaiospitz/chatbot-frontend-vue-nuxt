<template>
  <div class="chat">
    <div class="header">
      <span class="title">YodaBot</span>
    </div>

    <div id="chatBody" class="body">
      <div
        v-for="item in messages"
        :id="JSON.parse(item).id"
        :key="item"
        class="msg_wrapper"
        :class="JSON.parse(item).sender === 'bot' ? 'msg_bot' : 'msg_own'"
      >
        <!-- eslint-disable vue/no-v-html -->
        <span
          v-if="JSON.parse(item).listResponse === true"
          class="text"
          v-html="JSON.parse(item).message"
        >
          <!-- eslint-enable -->
        </span>

        <span v-if="JSON.parse(item).listResponse === false" class="text">
          {{ JSON.parse(item).message }}
        </span>

        <img
          v-if="JSON.parse(item).sender === 'bot'"
          class="picture"
          src="~assets/images/profile.jpeg"
          alt="YodaBot"
        />
      </div>

      <div v-if="loading" class="chatTypingWrapper">
        <img class="picture" src="~assets/images/profile.jpeg" alt="YodaBot" />

        <div class="dotsWrapper">
          <div class="dot-pulse"></div>
        </div>
      </div>
    </div>

    <form @submit.prevent="sendMessage">
      <input
        id="message"
        v-model="message"
        name="message"
        placeholder="Write your message..."
        required
      />
      <img src="~assets/icons/send.svg" />
    </form>
  </div>
</template>

<script>
const randomString = require('randomstring')

export default {
  name: 'ChatBoxComponent',
  data() {
    return {
      loading: false,
      sessionId: '',
      message: '',
      messages: [],
    }
  },
  watch: {
    sessionId(newSessionId) {
      localStorage.sessionId = newSessionId
    },
    messages(newMessages) {
      localStorage.messages = JSON.stringify(newMessages)
      this.scrollToBottomOfChat()
    },
    loading() {
      this.scrollToBottomOfChat()
    },
  },
  mounted() {
    if (localStorage.sessionId) {
      this.sessionId = localStorage.sessionId
    }
    if (localStorage.messages) {
      this.messages = JSON.parse(localStorage.messages)
    }
    window.addEventListener('load', (event) => {
      this.scrollToBottomOfChat()
    })
  },
  methods: {
    sendMessage() {
      if (this.message) {
        this.loading = true
        this.addMessage(this.message, 'user', false)
        this.$axios
          .post('https://api.kaiospitz.com.br/messages', {
            message: this.message,
            sessionId: this.sessionId,
          })
          .then((response) => {
            this.success = true
            this.errored = false
            if (response.data.sessionId) {
              this.sessionId = response.data.sessionId
            }
            if (response.data.botResponse) {
              this.addMessage(
                response.data.botResponse,
                'bot',
                response.data.listResponse
              )
            }
          })
          .catch((error) => {
            if (error) {
              this.errored = true

              try {
                if (error.response.data.errorCode) {
                  this.$notify({
                    group: 'all',
                    title: 'Something went wrong',
                    text: 'errorCode: ' + error.response.data.errorCode,
                    type: 'error',
                  })
                }
              } catch {
                this.$notify({
                  group: 'all',
                  title: 'Something went wrong',
                  text: 'Server unreachable',
                  type: 'error',
                })
              }
            }
          })
          .finally(() => {
            this.loading = false
            this.scrollToBottomOfChat()
          })
      }
      this.message = ''
    },
    addMessage(msg, msgSender, isListResponse) {
      const msgObject = JSON.stringify({
        id: randomString.generate(6),
        message: msg,
        sender: msgSender,
        listResponse: isListResponse,
      })
      this.messages.push(msgObject)
      this.scrollToBottomOfChat()
    },
    scrollToBottomOfChat() {
      this.$nextTick(() => {
        const bodyContainer = this.$el.querySelector('#chatBody')
        bodyContainer.scrollTop = bodyContainer.scrollHeight
      })
    },
  },
}
</script>
