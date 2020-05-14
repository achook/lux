<template>
  <div>
    <input v-model="color" @change="emitChange" type="color"/>
    <p> {{ color }} </p>
  </div>
</template>

<script>
export default {
  name: 'ColorPicker',

  data () {
    return {
      color: '#ff0000'
    }
  },

  mounted () {
    this.emitChange()
  },

  methods: {
    emitChange () {
      const color = this.color.substr(1).toUpperCase()
      this.$emit('change', color)
    }
  }
}
</script>

<style lang="scss">
</style>
