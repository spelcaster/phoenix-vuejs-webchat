<template>
  <div class="my-app">
        <h1>Vuechat</h1>
        <div class="user-details" v-if="enterName">
          <label for="username">Please enter your name:</label><br>
          <input type="text" name="username" v-model="username">
          <button v-on:click="connectToChat">Next</button>
        </div>
        <div class="messages" v-if="!enterName">
          <ul v-for="message in messages">
            <li>
              <strong>{{ message.username }}</strong>: {{ message.body }}<br>
              <small>{{ message.received_at }}</small>
            </li>
          </ul>
          <strong>{{ username }}:</strong><br>
          <input type="text" v-model="message" v-on:keyup.13="sendMessage">
        </div>
    </div>
</template>

<style lang="sass">
.my-app {
  margin-left: auto;
  margin-right: auto;
  width: 800px;

  h1 {
    text-align: center;
  }
}
</style>

<script>
import {Socket} from 'phoenix'

export default {
  data: function () {
    return {
      socket: null,
      channel: null,
      messages: [],
      message: '',
      username: '',
      enterName: true
    }
  },
  methods: {
    sendMessage: function () {
      this.channel.push('new_msg', {body: this.message})
      this.message = ''
    },
    connectToChat: function () {
      this.socket = new Socket('/socket', {params: {username: this.username}})
      this.socket.connect()

      this.channel = this.socket.channel('room:lobby', {})
      this.channel.on('new_msg', payload => {
        payload.received_at = Date()
        this.messages.push(payload)
      })

      this.channel.join()
        .receive('ok', response => {
          this.enterName = false
          console.log('Joined succesfully', response)
        })
        .receive('error', response => {
          console.log('Unable to join', response)
        })


    }
  }
}
</script>
