<template>
  <div id="app"
       @touchmove.prevent>
    <v-header :seller="seller"></v-header>
  </div>
</template>

<script>
  import VHeader from 'components/v-header/v-header'
  import { getSeller } from 'api'

  export default {
    name: 'App',
    data() {
      return {
        seller: {}
      }
    },
    created() {
      this._getSeller()
    },
    methods: {
      _getSeller() {
        getSeller().then((seller) => {
          this.seller = seller
        })
      }
    },
    components: {
      VHeader
    }
  }
</script>

<style lang="stylus" scoped></style>
