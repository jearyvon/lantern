<template>
<!--   <div  @scroll.native=""
        class="scroll-wrap"
        ref="wrap"
        :style="{
          'padding-right': style.barWidth + 'px',
          'padding-bottom': style.barHeight + 'px'
        }"
  > -->
    <div  class="lt-scroll__container"
          ref="wrap"
          @scroll.stop=""
          :style="{
            'width': style.wrapWidth,
            'height': style.wrapHeight,
            'padding-right': style.barWidth + 'px',
            'padding-bottom': style.barHeight + 'px'
          }"
    >
      <div class="lt-scroll__content" ref="inner"
          @scroll.stop="setScrollLeftTop"
      >
        <slot
          @scrollReload="scrollReload"
        ></slot>
      </div>
      <div  class="lt-scroll__track lt-scroll__track--right"
            v-if="rightBarShow"
            :style="{
              'background-color': style.trackColor,
              'width': style.barWidth + 'px',
              'bottom': 0 + 'px',
              'zIndex': style.zIndex,
              'border-radius': style.barWidth / 2 + 'px'
            }"
            :class="{'lt-scroll__track--auto-hide': style.autoHide && !barMousedown}"
            @mousedown.stop="rightMousedown"
      >
        <div  class="lt-scroll__bar lt-scroll__bar--right"
              :class="{'lt-scroll__bar--active': rightActive}"
              :style="{
              'background-color': style.barColor,
              'width': style.barWidth + 'px',
              'opacity': style.barOpacityInactive,
              'height': rightBarHeight + 'px',
              'top': barTop + 'px'
              }"
        ></div>
      </div>
      <div  class="lt-scroll__track lt-scroll__track--bottom"
            v-if="bottomBarShow"
            :style="{
              'background-color': style.trackColor,
              'height': style.barHeight + 'px',
              'right': 0 + 'px',
              'zIndex': style.zIndex,
              'border-radius': style.barHeight / 2 + 'px'
            }"
            :class="{'lt-scroll__track--auto-hide': style.autoHide && !barMousedown}"
            @mousedown.stop="bottomMousedown"
      >
        <div  class="lt-scroll__bar lt-scroll__bar--bottom"
              :class="{'lt-scroll__bar--active': bottomActive}"
              :style="{
                'background-color': style.barColor,
                'width': bottomBarWidth + 'px',
                'opacity': style.barOpacityInactive,
                'height': style.barHeight + 'px',
                'left': barLeft + 'px'
              }"
        ></div>
      </div>
    </div>
  <!-- </div> -->
</template>

<style lang="less">
  @import "../../../styles/packages/scroll.less";
</style>

<script>
export default {
  name: 'Scroll',
  data () {
    return {
      // 滚动条默认样式
      style: {
        barColor: '#959595',
        barWidth: 8,
        barHeight: 8,
        barOpacityInactive: 0.3,
        trackColor: '#eee',
        zIndex: 'auto',
        wrapWidth: '100%',
        wrapHeight: '100%',
        autoHide: true
      },
      // 是否显示右侧滚动条
      rightBarShow: false,
      // 是否显示下侧滚动条
      bottomBarShow: false,
      // 滚动条上触发mousedown事件
      barMousedown: false,
      // 右侧滚动条激活状态
      rightActive: false,
      // 下侧滚动条激活状态
      bottomActive: false,
      activeClock: null,
      rightActiveClock: null,
      bottomActiveClock: null,
      activeClockDelay: 800,

      wrapSize: {},

      // 容器的宽高
      wrapHeight: 0,
      wrapWidth: 0,
      // 内容的宽高
      innerHeight: 0,
      innerWidth: 0,
      // 滚动条的宽高
      rightBarHeight: 0,
      bottomBarWidth: 0,

      // 内容的滚动距离
      scrollTop: 0,
      scrollLeft: 0
    }
  },
  props: {
    barStyle: {
      required: false,
      type: Object
    }
  },
  mounted() {
    console.log(this.barStyle)
    Object.assign(this.style, this.barStyle);
    this.$nextTick(() => {
      this.scrollReload();
    });
  },
  watch: {

  },
  computed: {
    barTop() {
      let percent = this.scrollTop / (this.innerHeight - this.wrapHeight);
      return (this.wrapHeight - this.rightBarHeight) * percent;
    },
    barLeft() {
      let percent = this.scrollLeft / (this.innerWidth - this.wrapWidth);
      return (this.wrapWidth - this.bottomBarWidth) * percent;
    }
  },
  methods: {
    scrollReload() {
      this.getPercent();
    },
    getPercent() {
      const wrapNode = this.$refs.wrap;
      const slotnode = this.$refs.inner;
      if(this.barStyle.wrapWidth.includes('px')) {
        this.wrapWidth = parseInt(this.barStyle.wrapWidth) - this.style.barWidth;
      } else {
        this.wrapWidth = wrapNode.clientHeight - this.style.barWidth;
      }
      if(this.barStyle.wrapHeight.includes('px')) {
        this.wrapHeight = parseInt(this.barStyle.wrapHeight) - this.style.barHeight;
      } else {
        this.wrapHeight = wrapNode.clientHeight - this.style.barHeight;
      }
      // this.wrapHeight = wrapNode.clientHeight - this.style.barHeight;
      // this.wrapWidth = wrapNode.clientWidth - this.style.barWidth;
      this.innerHeight = slotnode.scrollHeight;
      this.innerWidth = slotnode.scrollWidth;
      if(this.wrapHeight < this.innerHeight) {
        this.rightBarHeight = this.wrapHeight / this.innerHeight * this.wrapHeight;
        this.rightBarShow = true;
      }
      if(this.wrapWidth < this.innerWidth) {
        this.bottomBarWidth = this.wrapWidth / this.innerWidth * this.wrapWidth;
        this.bottomBarShow = true;
      }
    },
    setScrollLeftTop(eve) {
      let node = eve ? eve.target : this.$refs.inner;
      if(this.scrollTop != node.scrollTop) {
        this.rightActive = true;
      }
      if(this.scrollLeft != node.scrollLeft) {
        this.bottomActive = true;
      }
      this.scrollLeft = node.scrollLeft;
      this.scrollTop = node.scrollTop;
      if(!this.barMousedown) {
        clearTimeout(this.activeClock);
        this.activeClock = setTimeout(() => {
          this.rightActive = false;
          this.bottomActive = false;
        }, this.activeClockDelay);
      }
    },
    rightMousedown(eve) {
      eve.preventDefault();
      let startY = eve.clientY,
          nowY,
          move,
          newTop,
          percent,
          scrollTop,
          oldTop = this.barTop;
      let nodeMousemove = (eve) => {
        nowY = eve.clientY;
        move = nowY - startY;
        newTop = oldTop + move;
        newTop = newTop < 0 ? 0 : newTop;
        newTop = newTop > this.wrapHeight - this.rightBarHeight ? this.wrapHeight - this.rightBarHeight : newTop;
        percent = newTop / (this.wrapHeight - this.rightBarHeight);
        scrollTop = (this.innerHeight - this.wrapHeight) * percent;
        this.$refs.inner.scrollTop = scrollTop;
        this.scrollTop = scrollTop;
      }

      this.barMousedown = true;
      clearTimeout(this.rightActiveClock);
      this.rightBarShow = true;

      let nodeMouseup = () => {
        window.removeEventListener('mousemove', nodeMousemove);
        window.removeEventListener('mouseup', nodeMouseup);
        this.barMousedown = false;
        this.rightActiveClock = setTimeout(() => {
          this.rightActive = false;
        }, this.activeClockDelay);
      }

      window.addEventListener('mousemove', nodeMousemove);
      window.addEventListener('mouseup', nodeMouseup);
    },
    bottomMousedown(eve) {
      eve.preventDefault();
      let startX = eve.clientX,
          nowX,
          move,
          newLeft,
          percent,
          scrollLeft,
          oldLeft = this.barLeft;
      let nodeMousemove = (eve) => {
        nowX = eve.clientX;
        move = nowX - startX;
        newLeft = oldLeft + move;
        newLeft = newLeft < 0 ? 0 : newLeft;
        newLeft = newLeft > this.wrapWidth - this.bottomBarWidth ? this.wrapWidth - this.bottomBarWidth : newLeft;
        percent = newLeft / (this.wrapWidth - this.bottomBarWidth);
        scrollLeft = (this.innerWidth - this.wrapWidth) * percent;
        this.$refs.inner.scrollLeft = scrollLeft;
        this.scrollLeft = scrollLeft;
      }

      this.barMousedown = true;
      clearTimeout(this.bottomActiveClock);
      this.bottomActive = true;

      let nodeMouseup = () => {
        window.removeEventListener('mousemove', nodeMousemove);
        window.removeEventListener('mouseup', nodeMouseup);
        this.barMousedown = false;
        this.bottomActiveClock = setTimeout(() => {
          this.bottomActive = false;
        }, this.activeClockDelay);
      }

      window.addEventListener('mousemove', nodeMousemove);
      window.addEventListener('mouseup', nodeMouseup);
    }
  }
}
</script>