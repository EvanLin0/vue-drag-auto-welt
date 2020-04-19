<!--
  -- 基于 vue 的可拖动的悬浮球，自动贴边
  -- 支持属性：value 事件 @click 插槽 value
  -- 使用示例：<drag-ball :value="message" @click="alert('悬浮球事件')"><div slot="value">悬浮球！</div></drag-ball>
  -- npm i vue-drag-ball
  -- import dragBall from 'vue-drag-ball'
  -- Vue.use(dragBall)
-->

<template>
<div class="drag-ball"
     ref="dragBall"
     @touchstart.stop.prevent="touchstart"
     @touchmove.stop.prevent="touchmove"
     @touchend.stop.prevent="touchend"
>
  <div class="drag-content">
    <slot name="value">{{value}}</slot>
  </div>
</div>
</template>

<script>
export default {
  name: 'drag-ball',
  props: {
    value: {
      type: String,
      default: '悬浮球!'
    }
  },
  data () {
    return {
      canDrag: false,
      inset: { // 偏移
        left: 0,
        top: 0
      },
      // 移动
      move: {},
      // 位置
      position: {
        left: 0,
        top: 0
      },
      positionOld: {}, // 初始位置
      startTime: null // touchstart 按下的时间戳
    }
  },
  computed: {
    dragBall () { // 把 this.$refs.dragBll 映射成 this.dragBall
      return this.$refs.dragBall
    }
  },
  methods: {
    toNew () {
      alert('去新版')
    },
    touchstart (e) {
      if (!this.canDrag) {
        console.log(e)
        console.log(this.$refs.dragBall.offsetLeft)
        this.startTime = e.timeStamp // 保存时间戳
        // 记录初始位置
        this.positionOld = this.getPosition(this.dragBall)
        // 浅复制一份作为位置信息
        this.position = {...this.positionOld}
        // 偏移
        this.inset = {
          left: e.targetTouches[0].clientX - this.positionOld.left,
          top: e.targetTouches[0].clientY - this.positionOld.top
        }
        this.canDrag = true
      }
    },
    touchmove (e) {
      if (this.canDrag) {
        let left = e.targetTouches[0].clientX - this.inset.left
        let top = e.targetTouches[0].clientY - this.inset.top
        if (left < 0) {
          left = 0
        } else if (left > (window.innerWidth - this.dragBall.offsetWidth)) {
          left = window.innerWidth - this.dragBall.offsetWidth
        }
        if (top < 0) {
          top = 0
        } else if (top > (window.innerHeight - this.dragBall.offsetHeight)) {
          top = window.innerHeight - this.dragBall.offsetHeight
        }
        this.dragBall.style.left = left + 'px'
        this.dragBall.style.top = top + 'px'
        this.move = {
          x: left - this.positionOld.left,
          y: top - this.positionOld.top
        }
        this.position = {left, top}
      }
    },
    touchend (e) {
      if (this.canDrag) {
        this.endTime = e.timeStamp
        if (this.endTime - this.startTime > 400 || Math.abs(this.move.x) > 2 || Math.abs(this.move.y) > 2) {
          // 非单击事件
          if ((this.position.left + this.dragBall.offsetWidth / 2) > window.innerWidth / 2) {
            this.dragBall.style.left = window.innerWidth - this.dragBall.offsetWidth + 'px'
          } else {
            this.dragBall.style.left = 0 + 'px'
          }
        } else {
          this.$emit('click')
        }
        this.inset = {}
        this.move = {}
        this.position = {}
        this.canDrag = false
      }
    },
    /**
     * @description 获取 DOM 的绝对位置
     * @author LinW (evan_origin@163.com)
     * @date 2020-04-18 23:11
     * @param {Function}
     * @returns {number}
     */
    getPosition (source) {
      let left = source.offsetLeft // 获取元素相对于其父元素的left值
      let top = source.offsetTop
      let current = source.offsetParent // 取得元素的offsetParent // 一直循环直到根元素
      while (current != null) {
        left += current.offsetLeft
        top += current.offsetTop
        current = current.offsetParent
      }
      return {
        left: left,
        top: top
      }
    }
  }
}
</script>

<style scoped>
  .drag-ball {
    position: absolute;
    z-index: 10003;
    right: 0;
    top: 70%;
    width: 5em;
    height: 5em;
    background: deepskyblue;
    border-radius: 50%;
    overflow: hidden;
    box-shadow: 0px 0px 10px 2px skyblue;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 1em;
    user-select: none;
  }
  .drag-ball .drag-content {
    overflow-wrap: break-word;
    font-size: 14px;
    color: #fff;
    letter-spacing: 2px;
  }
</style>
