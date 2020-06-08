<template>
  <div>
    <input v-model="internalColor" type="color"/>
  </div>
</template>

<script>
export default {
  name: 'ColorPicker',

  props: {
    color: String
  },

  computed: {
    internalColor: {
      get () {
        return this.color
      },

      set (value) {
        this.$emit('update:color', value)
      }

    }
  }
}
</script>

<style lang="scss">
</style>
