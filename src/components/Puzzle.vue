<template>
  <div class="puzzle">
    <div class="puzzle-wrapper"
      :style="{ width: width + 'px', height: height + 'px' }">
      <div
        v-for="(grid, index) in grids"
        :key="index"
        class="grid"
        :style="{
          ...gridStyle,
          backgroundPosition: grid.style.backgroundPosition,
          left: grid.style.left,
          top: grid.style.top
        }"
        @click="handleClickGrid(grid)"></div>
    </div>
    <img width="300" height="300" :src="imgSrc" />
  </div>
</template>

<script>
const difficulties = [9, 16, 25, 36]

export default {
  props: {
    width: {
      type: Number,
      default: 300
    },
    height: {
      type: Number,
      default: 300
    },
    imgSrc: {
      type: String,
      required: true
    },
    // TODO 过滤不合法值
    difficulty: {
      type: Number,
      default: 1
    }
  },
  data () {
    return {
      grids: [],
      blankPosition: {}
    }
  },
  computed: {
    // 总方格数
    size () {
      return difficulties[this.difficulty - 1]
    },
    // 行/列方格数
    lineCount () {
      return Math.sqrt(this.size)
    },
    gridWidth () {
      return this.width / this.lineCount
    },
    gridHeight () {
      return this.height / this.lineCount
    },
    gridStyle () {
      return {
        width: this.gridWidth + 'px',
        height: this.gridHeight + 'px',
        backgroundImage: `url(${this.imgSrc})`,
        backgroundSize: `${this.lineCount * 100}% ${this.lineCount * 100}%`,
      }
    },
    // 空白方格周围可移动方格列表
    legalGrids () {
      const { row, col } = this.blankPosition, { lineCount } = this
      let list = [
        { row, col: col - 1 },
        { row, col: col + 1 },
        { row: row - 1, col },
        { row: row + 1, col }
      ]
      list = list.filter(({ row, col }) => row >= 0 && col >= 0 && row < lineCount && col < lineCount)
      return list
    },
    // 是否完成
    // TODO 完成某一行/列特效
    completed () {
      return this.grids.every(({ position: pos, currentPosition: curr }) => pos.row === curr.row && pos.col === curr.col)
    }
  },
  watch: {
    completed (value) {
      if (value) {
        console.log('completed')
        this.$emit('completed')
      }
    }
  },
  mounted() {
    this.generate()
  },
  methods: {
    generate () {
      this.initial()
      this.randomGrids()
    },
    // 初始化拼图
    initial () {
      const { lineCount, gridWidth, gridHeight } = this
      for (let row = 0; row < lineCount; row++) {
        for (let col = 0; col < lineCount; col++) {
          this.grids.push({
            index: row * lineCount + col,
            // 原始位置
            position: {
              row,
              col
            },
            // 当前位置
            currentPosition: {
              row,
              col
            },
            style: {
              left: col * gridWidth + 'px',
              top: row * gridHeight + 'px',
              backgroundPosition: `${col * (100 / (lineCount - 1))}% ${row * (100 / (lineCount - 1))}%`
            }
          })
        }
      }
      // 移除右下角方格
      this.grids.splice(this.size - 1)
      // 记录空白方格位置
      this.blankPosition = {
        row: this.lineCount - 1,
        col: this.lineCount - 1
      }
    },
    // 打乱拼图
    randomGrids () {
      // 避免没有打乱
      while (this.completed) {
        let times = this.size * 3
        while (times > 0) {
          this.randomStep()
          times--
        }
      }
    },
    randomStep () {
      const { legalGrids, grids } = this
      const length = legalGrids.length
      const randomIndex = Math.floor(Math.random() * length)
      let randomGrid = legalGrids[randomIndex]
      randomGrid = grids.find(({ currentPosition: pos }) => pos.row === randomGrid.row && pos.col === randomGrid.col)
      if (randomGrid) {
        this.handleClickGrid(randomGrid)
      }
    },
    handleClickGrid (grid) {
      const { row, col } = grid.currentPosition
      
      // 点击方格是否可移动
      const findIndex = this.legalGrids.findIndex(grid => grid.row === row && grid.col === col)
      if (findIndex >= 0) {
        // 移动当前空格
        const grids = [ ...this.grids ]
        const currentGrid = grids[grid.index]
        const newRow = this.blankPosition.row
        const newCol = this.blankPosition.col
        currentGrid.currentPosition = {
          row: newRow,
          col: newCol
        }
        currentGrid.style.left = newCol * this.gridWidth + 'px'
        currentGrid.style.top = newRow * this.gridHeight + 'px'
        this.grids = grids
        // 改变空白位置
        this.blankPosition = {
          row,
          col
        }

      } else {
        // TODO 抖动效果
      }
    }
  }
}
</script>

<style scoped>
.puzzle-wrapper {
  position: relative;
}
.puzzle-wrapper .grid {
  position: absolute;
  background-repeat: no-repeat;
  transition: all .2s;
}
</style>
