<template>
  <div>
    <puzzle :imgSrc="require('../assets/puzzle.jpg')" :difficulty="2"></puzzle>
  </div>
</template>

<script>
import Puzzle from './Puzzle'

export default {
  components: {
    Puzzle
  },
  data () {
    return {

    }
  }
}
</script>

<style scoped>

</style>
