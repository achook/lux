<template>
  <div id="app">
    <h1>LED</h1>
      <div class="buttons">
        <button @click="setMode(1)">Kolor</button>
        <button @click="setMode(2)">Miganie</button>
        <button @click="setMode(3)">Fala</button>
        <button @click="setMode(4)">Cykl</button>
        <button @click="setMode(0)">Wyłącz</button>
      </div>
      <ColorPicker v-if="picker" :color.sync="color"/>
  </div>
</template>

<script>
import ColorPicker from './components/ColorPicker'

const MQTT_URL = 'mqtt.server'
const MQTT_PORT = 9001
const MQTT_DEVICE = 'lux'
const MQTT_USER = 'led_client'
const MQTT_PASSWORD = 't7kXRroFJwa5Qf5G'
const MQTT_CHANNEL = 'device/2/mode'
const MQTT_CALLBACK_CHANNEL = 'device/2/status'

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

      mode: 0,
      color: '#00FF00',

      picker: false
    }
  },

  created () {
    this.client = new mqtt.Client(MQTT_URL, MQTT_PORT, '', MQTT_DEVICE)

    this.client.onConnectionLost = () => {
      this.connected = false
      this.connect()
    }

    this.client.onMessageArrived = (message) => {
      console.log(message)

      const hexMode = message.payloadString.substr(0, 2)
      const hexColor = message.payloadString.substr(3)

      this.mode = parseInt(hexMode, 16)
      this.color = `#${hexColor}`

      this.client.onMessageArrived = () => {}
    }

    this.connect()
  },

  computed: {
    strippedColor () {
      return this.color.toUpperCase().substr(1)
    },

    hexMode () {
      return ('0' + this.mode.toString(16)).substr(-2)
    }
  },

  watch: {
    color () {
      this.send(this.hexMode, this.strippedColor)
    },

    mode () {
      if (this.mode < 3) this.picker = true
      else this.picker = false

      this.send(this.hexMode, this.strippedColor)
    }
  },

  methods: {
    connect () {
      this.client.connect({
        userName: MQTT_USER,
        password: MQTT_PASSWORD,
        onSuccess: () => {
          this.connected = true
          this.client.subscribe(MQTT_CALLBACK_CHANNEL)
        },
        onFailure: (context, code, message) => { this.throwError(message) }
      })
    },

    send (code, color) {
      if (!this.connected) return

      const opcode = `${code}:${color}`
      console.log(opcode)

      const message = new mqtt.Message(opcode)
      message.destinationName = MQTT_CHANNEL
      this.client.send(message)
    },

    setMode (mode) {
      this.mode = mode
    },

    enablePicker () {
      this.picker = true
    },

    disablePicker () {
      this.picker = false
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
