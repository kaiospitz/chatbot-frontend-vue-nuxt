<template>
  <div class="main">
    <div class="chat">
      <div class="header">
        <span class="title">YodaBot</span>
      </div>

      <div class="body">
        <div class="msg_wrapper msg_own">
          <span class="text">Own Message</span>
        </div>

        <div class="msg_wrapper msg_bot">
          <span class="text">
            Bot Message Bot Message Bot Message Bot Message Bot Message Bot
            Message Bot Message Bot Message Bot Message Bot Message Bot Message
            Bot Message
          </span>
          <img
            class="picture"
            src="~assets/images/profile.jpeg"
            alt="YodaBot"
          />
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
  </div>
</template>

<script>
export default {
  name: 'HomeComponent',
  data() {
    return {
      message: '',
    }
  },
  methods: {
    sendMessage() {
      this.loading = true
      this.$axios
        .post('/messages', {
          message: this.message,
        })
        .then((response) => {
          this.success = true
          this.errored = false
        })
        .catch((error) => {
          if (error) {
            this.errored = true
          }
        })
        .finally(() => {
          this.loading = false
        })
    },
  },
}
</script>
