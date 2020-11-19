<template>
    <div>
        <puzzle :imgSrc="require('../assets/puzzle.png')" :difficuty="3"></puzzle>
    </div>
</template>

<script>
import Puzzle from './Puzzle'

export default {
    components: {
        Puzzle
    },
    data() {
        return {

        }
    }
}
</script>

<style scoped>

</style>
