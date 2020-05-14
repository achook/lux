<template>
  <div id="app">
    <h1>LED</h1>
      <div class="buttons">
        <button @click="set(1)">Kolor</button>
        <button @click="set(2)">Miganie</button>
        <button @click="set(3)">Fala</button>
        <button @click="set(4)">Cykl</button>
        <button @click="set(0)">Wyłącz</button>
      </div>
      <ColorPicker @change="updateColor" />
  </div>
</template>

<script>
import ColorPicker from './components/ColorPicker'

const MQTT_URL = 'malinka.local'
const MQTT_PORT = 9001
const MQTT_DEVICE = 'lux'
const MQTT_USER = 'sezamek'
const MQTT_PASSWORD = 'kANQcGkgeg6rlZnk'
const MQTT_CHANNEL = 'test'

const mqtt = require('paho-mqtt')

export default {
  name: 'App',

  components: {
    ColorPicker
  },

  data () {
    return {
      client: null,
      connected: false,
      color: 'FFFFFF'
    }
  },

  created () {
    this.client = new mqtt.Client(MQTT_URL, MQTT_PORT, '', MQTT_DEVICE)
    this.client.connect({
      userName: MQTT_USER,
      password: MQTT_PASSWORD,
      onSuccess: () => { this.connected = true },
      onFailure: (context, code, message) => { this.throwError(message) }
    })
  },

  methods: {
    set (code) {
      if (!this.connected) return

      const message = new mqtt.Message(code.toString())
      message.destinationName = MQTT_CHANNEL
      this.client.send(message)
    },

    updateColor (color) {
      this.color = color
    },

    throwError (message) {
      console.log(message)
    }
  }
}
</script>

<style lang="scss">
body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', 'Open Sans', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;

  text-align: center;
}
</style>
