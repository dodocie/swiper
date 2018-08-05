<template>
  <div id="swiper">
    <div class="wrapperbox">
      <ul class="wrapper" ref="movebtn" @touchstart="down" @touchmove.prevent="move"
          @touchend="end">
        <li :style="{marginLeft: marginLR, marginRight: marginLR, width: itemWidth, height: itemHeight}" v-for="(item, i) in itemList" v-html="i" :key="i" :class="{activeItem: i===currIndex}"></li>
      </ul>
    </div>
  </div>
</template>

<script>
  /* eslint-disable */
  export default {
    name   : 'swiper',
    data() {
      return {
        itemLen : 0,
        positionX: 0,
        moveX    : 0,
        originX  : 0,
        stopX    : 0,
        flags    : false,
        currIndex: 0,
        nodeWidth: 0,
        spaceX   : 0,
        distance : 0,
        maxLeft  : 0,
        maxRight : 0,
        facNum : 0
      }
    },
    props  : {
      showMax: {
        type: Number,
        default: 3
      },
      marginLR:{
        type: String,
        default: '1rem'
      },
      itemWidth:{
        type: String,
        default: '5rem'
      },
      itemHeight:{
        type: String,
        default: '5rem'
      },
      itemList: {
        type: Array,
        default(){
          return [0,1,2,3,4,5,6,7,8,9]
        }
      }
    },
    mounted() {
      let showMax = this.showMax,
          facNum = Math.floor(showMax/2)
      let box     = document.querySelector('.wrapper'),
          node = box.childNodes[0],
          itemWidth = node.clientWidth

      this.itemLen   = this.itemList.length
      this.spaceX    = node.offsetLeft * 2

      this.distance  = this.spaceX + itemWidth
      box.style.left = showMax >=3 ? `${this.distance * facNum}px` : 0
      document.querySelector('.wrapperbox').style.width = `${showMax * this.distance}px`

      console.log('distance', this.distance)
      this.maxLeft  = -this.distance *(this.itemLen - Math.ceil(showMax/2))
      this.maxRight = this.distance * facNum
      this.facNum = facNum
      console.log(this.maxLeft)
      console.log(this.maxRight)

      this.activeItem()
    },
    watch  : {
      currIndex(val) {
        console.log('index: ', val)
        this.$emit('onTransitionEnd', val)
      }
    },
    methods: {
      activeItem() {
        let box     = document.querySelector('.wrapper'),
            left    = box.style.left || '0',
            boxLeft = parseInt(left)

        if (this.itemList.length > 1) {
          if (boxLeft < 0) {
            this.currIndex = Math.floor(Math.abs(boxLeft) / this.distance) + this.facNum
            this.currIndex >= this.itemLen && (this.currIndex = this.itemLen-1)
          } else if (boxLeft === 0) {
            this.currIndex = this.facNum
          } else {
            this.currIndex = this.facNum - Math.floor(boxLeft / this.distance)
            this.currIndex < 0 && (this.currIndex = 0)
          }
        } else {
          this.currIndex = 0
        }
      },
      down() {
        this.flags = true
        let touch
        if (event.touches) {
          touch = event.touches[0]
        } else {
          touch = event
        }

        this.positionX = touch.clientX
        this.originX   = this.$refs.movebtn.offsetLeft
      },
      move() {
        if (this.flags) {
          let touch
          if (event.touches) {
            touch = event.touches[0]
          } else {
            touch = event
          }
          let moveBtn = this.$refs.movebtn

          this.moveX = touch.clientX - this.positionX
          this.stopX = this.originX + this.moveX

          moveBtn.style.left = `${this.stopX}px`
          this.activeItem()
        }
      },
      end() {
        setTimeout(() => {
          this.flags = false
        }, 500)
        let x            = this.moveX,
            d            = this.distance
        let moveDistance = Math.round(x / d) * d
        let moveBtn      = this.$refs.movebtn
        this.stopX       = this.originX + moveDistance

        this.stopX >= this.maxRight && (this.stopX = this.maxRight)
        this.stopX <= this.maxLeft && (this.stopX = this.maxLeft)
        moveBtn.style.left = `${this.stopX}px`

        this.activeItem()
      }
    }
  }
</script>

<style scoped>
  #swiper {
    margin: 20px auto;
  }

  ul, li {
    list-style: none;
  }

  .wrapperbox {
    position: relative;
    overflow: hidden;
    -webkit-overflow-scrolling: touch;
    overflow-scrolling: touch;
    z-index: 1;
    margin: 0 auto;
    /*border: 1px solid #2c3e50;*/
  }

  ul {
    position: relative;
    transition: 500ms;
    transition-timing-function: ease;
    width: 100%;
    z-index: 1;
    display: flex;
    -webkit-overflow-scrolling: touch;
    overflow-scrolling: touch;
  }

  li {
    background-color: red;
    transition: 500ms;
    flex-shrink: 0;
    transform: scale(.8);
    border-radius: 50%;
    -webkit-overflow-scrolling: touch;
    overflow-scrolling: touch;
  }

  .activeItem {
    opacity: 1;
    transform: scale(1);
    -webkit-transform: scale(1);
    -webkit-overflow-scrolling: touch;
    overflow-scrolling: touch;
  }

  .move-btn {
    border-radius: 50%;
    width: 5rem;
    height: 5rem;
    background-color: #2c3e50;
    position: fixed;
    z-index: 999;
    bottom: 5rem;
    left: 2rem;
    -webkit-overflow-scrolling: touch;
    overflow-scrolling: touch;
  }

</style>
