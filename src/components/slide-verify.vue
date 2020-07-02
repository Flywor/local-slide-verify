<template>
  <div class="slide-verify" onselectstart="return false;">
    <div class="slide-verify-refresh-icon" @click="refresh">
      <span class="md-refresh" />
    </div>
    <canvas ref="canvas" :width="w" :height="h"></canvas>
    <canvas
      ref="block"
      :width="w"
      :height="h"
      class="slide-verify-block"
    ></canvas>
    <div
      class="slide-verify-slider"
      :class="{
        'container-active': containerActive,
        'container-success': containerSuccess,
        'container-fail': containerFail,
      }"
      :style="{ width: `${w}px` }"
    >
      <div class="slide-verify-slider-mask" :style="{ width: sliderMaskWidth }">
        <div
          class="slide-verify-slider-mask-item"
          :style="{ left: sliderLeft }"
          @mousedown="handleMouseDown"
          @touchstart="handleMouseDown"
        >
          <span class="arrow-right"></span>
        </div>
      </div>
      <span class="slide-verify-slider-text">
        {{ containerFail ? failText: sliderText }}
      </span>
    </div>
  </div>
</template>
<script>
const PI = Math.PI
function sum (x, y) {
  return x + y
}
function square (x) {
  return x * x
}
export default {
  name: 'slide-verify',
  props: {
    l: {
      type: Number,
      default: 42
    },
    r: {
      type: Number,
      default: 10
    },
    w: {
      type: Number,
      default: 310
    },
    h: {
      type: Number,
      default: 155
    },
    sliderText: {
      type: String,
      default: '向右滑动'
    },
    failText: {
      type: String,
      default: '请重试'
    }
  },
  data() {
    return {
      containerActive: false,
      containerSuccess: false,
      containerFail: false,
      canvasCtx: null,
      blockCtx: null,
      block: null,
      block_x: undefined,
      block_y: undefined,
      L: this.l + this.r * 2 + 3,
      originX: undefined,
      originY: undefined,
      isMouseDown: false,
      trail: [],
      sliderLeft: 0,
      sliderMaskWidth: 0
    }
  },
  mounted () {
    this.init()
  },
  beforeDestroy () {
    this.handleEvents('remove')
  },
  methods: {
    init () {
      this.initDom()
      this.initImg()
      this.handleEvents('add')
    },
    initDom () {
      this.block = this.$refs.block
      this.canvasCtx = this.$refs.canvas.getContext("2d")
      this.blockCtx = this.block.getContext("2d")
    },
    initImg () {
      this.createImg()
      this.drawBlock()
      this.blockCtx.drawImage(this.$refs.canvas, 0, 0, this.w, this.h)
      let { block_x: x, block_y: y, r, L } = this
      let _y = y - r * 2 - 1
      let ImageData = this.blockCtx.getImageData(x, _y, L, L)
      this.block.width = L
      this.blockCtx.putImageData(ImageData, 0, _y)
    },
    drawBlock () {
      this.block_x = this.getRandomNumberByRange(
        this.L + 10,
        this.w - (this.L + 10)
      )
      this.block_y = this.getRandomNumberByRange(
        10 + this.r * 2,
        this.h - (this.L + 10)
      )
      this.draw(this.canvasCtx, this.block_x, this.block_y, 'fill')
      this.draw(this.blockCtx, this.block_x, this.block_y, 'clip')
    },
    draw (ctx, x, y, operation) {
      let { l, r } = this
      ctx.beginPath()
      ctx.moveTo(x, y)
      ctx.arc(x + l / 2, y - r + 2, r, 0.72 * PI, 2.26 * PI)
      ctx.lineTo(x + l, y)
      ctx.arc(x + l + r - 2, y + l / 2, r, 1.21 * PI, 2.78 * PI)
      ctx.lineTo(x + l, y + l)
      ctx.lineTo(x, y + l)
      ctx.arc(x + r - 2, y + l / 2, r + 0.4, 2.76 * PI, 1.24 * PI, true)
      ctx.lineTo(x, y)
      ctx.lineWidth = 2
      ctx.fillStyle = `rgba(255, 255, 255, .8)`
      ctx.strokeStyle = `rgba(255, 255, 255, .8)`
      ctx.stroke()
      ctx[operation]()
      ctx.globalCompositeOperation = 'xor'
    },
    createImg () {
      const context = this.canvasCtx
      const getBackgroud = (Transparency) => {
        const RColor = Math.floor(255 * Math.random())
        const GColor = Math.floor(255 * Math.random())
        const BColor = Math.floor(255 * Math.random())
        return `rgba(${RColor}, ${GColor}, ${BColor}, ${Transparency})`
      }
      context.fillStyle = getBackgroud(.6)
      context.fillRect(0, 0, this.w, this.h)
      for (let i = 0; i < 20; i++) {
        context.strokeStyle = getBackgroud(1)
        context.lineWidth = Math.random() * 3
        context.beginPath()
        context.moveTo(Math.floor(Math.random() * this.w), Math.floor(Math.random() * this.h))
        context.lineTo(Math.floor(Math.random() * this.w), Math.floor(Math.random() * this.h))
        context.stroke()
      }
      for (let i = 0; i < 20; i++) {
        const px = Math.floor(Math.random() * this.w)
        const py = Math.floor(Math.random() * this.h)
        context.beginPath()
        context.lineWidth = Math.random() * 3
        context.strokeStyle = getBackgroud(1)
        context.arc(px, py, Math.random() * 10, Math.PI * 2, false)
        context.fillStyle = getBackgroud(1)
        context.fill()
        context.stroke()
      }
    },
    getRandomNumberByRange (start, end) {
      return Math.round(Math.random() * (end - start) + start)
    },
    refresh () {
      this.reset()
      this.$emit('refresh')
    },
    handleEvents(operation) {
      document[`${operation}EventListener`]('mousemove', this.handleMouseMove)
      document[`${operation}EventListener`]('mouseup', this.handleMouseUp)
      document[`${operation}EventListener`]('touchmove', this.handleMouseMove)
      document[`${operation}EventListener`]('touchend', this.handleMouseUp)
    },
    handleMouseDown (e) {
      this.originX = this.getPosition(e, 'X')
      this.originY = this.getPosition(e, 'Y')
      this.isMouseDown = true
    },
    handleMouseMove (e) {
      if (!this.isMouseDown) return false
      const moveX = this.getPosition(e, 'X') - this.originX
      const moveY = this.getPosition(e, 'Y') - this.originY
      if (moveX < 0 || moveX + 38 >= this.w) return false
      this.sliderLeft = `${moveX}px`
      let blockLeft = ((this.w - 40 - 20) / (this.w - 40)) * moveX
      this.block.style.left = `${blockLeft}px`
      this.containerActive = true
      this.sliderMaskWidth = `${moveX}px`
      this.trail.push(moveY)
    },
    handleMouseUp (e) {
      if (!this.isMouseDown) return false
      this.isMouseDown = false
      if (this.getPosition(e, 'X') === this.originX) return false
      this.containerActive = false
      const { spliced, TuringTest } = this.verify()
      if (spliced) {
        if (TuringTest) {
          this.containerSuccess = true
          this.$emit('success')
        } else {
          this.containerFail = true
          this.reset()
        }
      } else {
        this.containerFail = true
        this.$emit('fail')
        setTimeout(() => {
          this.reset()
        }, 1000)
      }
    },
    getPosition (e, direct) {
      return e.changedTouches ?
        e.changedTouches[0][`page${direct}`]
        :
        e[`client${direct}`]
    },
    verify () {
      const arr = this.trail
      const average = arr.reduce(sum) / arr.length
      const deviations = arr.map((x) => x - average)
      const stddev = Math.sqrt(deviations.map(square).reduce(sum) / arr.length)
      const left = parseInt(this.block.style.left)
      return {
        spliced: Math.abs(left - this.block_x) < 10,
        TuringTest: average !== stddev
      }
    },
    reset () {
      this.containerActive = false
      this.containerSuccess = false
      this.containerFail = false
      this.sliderLeft = 0
      this.block.style.left = 0
      this.sliderMaskWidth = 0
      const { w, h } = this
      this.canvasCtx.clearRect(0, 0, w, h)
      this.blockCtx.clearRect(0, 0, w, h)
      this.block.width = w
      this.initImg()
    }
  }
}
</script>
<style lang="less" scoped>
.slide-verify {
  display: inline-block;
  position: relative;
  &-block {
    position: absolute;
    top: 0;
    left: 0;
  }
  &-slider {
    position: relative;
    height: 40px;
    line-height: 40px;
    text-align: center;
    background: #f7f9fa;
    border: 1px solid #e4e7eb;
    &-mask {
      position: absolute;
      top: 0;
      left: 0;
      &-item {
        position: absolute;
        top: 0;
        left: 0;
        width: 38px;
        height: 38px;
        cursor: pointer;
        background: #2d8cf0;
        transition: background .2s, opacity .3s;
        .arrow-right {
          margin-top: 10px;
          margin-left: -7px;
          display: inline-block;
          border: 3px solid #fff;
          border-left: 0;
          border-top: 0;
          transform: rotate(-45deg);
          width: 16px;
          height: 16px;
        }
      }
      &-item:hover {
        opacity: .8;
      }
    }
    &:before {
      content: '';
      width: 100%;
      height: 100%;
      background: #01A2FA;
      position: absolute;
      left: 0;
      top: 0;
      opacity: .1;
      animation: lighter 2s infinite;
      @keyframes lighter {
        0% { width: 0; }
        50% { width: 100%; }
        100% { opacity: 0; }
      }
    }
  }
  &-refresh-icon {
    position: absolute;
    right: 35px;
    top: 5px;
    z-index: 1;
    cursor: pointer;
    .md-refresh {
      transition: opacity .3s;
      &:hover {
        opacity: .7;
      }
      &:before {
        content: '';
        width: 20px;
        height: 20px;
        border: 4px solid #fff;
        border-top: 4px solid transparent;
        border-radius: 100%;
        position: absolute;
        top: 7px;
        left: 0;
        transform: rotate(45deg);
        transition: transform .3s;
      }
      &:after {
        content: '';
        position: absolute;
        border: 9px solid transparent;
        border-left: 9px solid #fff;
        right: -31px;
        top: 2px;
      }
    }
  }
  .container-success {
    .slide-verify-slider-mask {
      background-color: #d2f4ef;
      &-item {
        background-color: #13ce66 !important;
      }
    }
  }
  .container-fail {
    .slide-verify-slider-mask {
      background-color: #fce1e1;
      &-item {
        background-color: #ff4d4f !important;
      } 
    }
  }
  .container-active .slide-verify-slider-text,
  .container-success .slide-verify-slider-text,
  .container-fail .slide-verify-slider-text {
    display: none;
  }
}
</style>