<template>
    <div
        @mouseover="showImage = true"
        @mouseout="showImage = true">
        <!-- puzzle的情况 -->
        <div v-if="type === '2'" class="verify-img-out"
             v-show="showImage"
             @mouseover="showImage = true"
             @mouseout="showImage = true"
             :style="{ height: (parseInt(setSize.imgHeight) + vSpace) + 'px' }"
        >
            <div class="verify-img-panel" :style="{width: setSize.imgWidth,
                                                   height: setSize.imgHeight,
                                                   background: 'url(' + defaultImg +')',
                                                   'background-size' : setSize.imgWidth + ' '+ setSize.imgHeight}">
                <div class="verify-refresh" @click="refresh"><i class="iconfont icon-refresh"></i>
                </div>
                <div class="verify-gap"
                     :style="{'width': blockSize.width, 'height': blockSize.height, top: top + 'px', left: left + 'px'}"></div>
            </div>
        </div>
        
        <!-- 公共部分 -->
        <div class="verify-bar-area" :style="{width: barSize.width,
                                              height: barSize.height,
                                              'line-height':barSize.height}">
            <span class="verify-msg" v-text="text"></span>
            <div class="verify-left-bar"
                 :style="{width: (leftBarWidth!==undefined)?leftBarWidth: barSize.height, height: barSize.height, 'border-color': leftBarBorderColor, transaction: transitionWidth}">
                <span class="verify-msg" style=" white-space:nowrap;" v-text="finishText"></span>
                <div class="verify-move-block"
                     @touchstart="start"
                     @mousedown="start"
                     :style="{width: barSize.height, height: barSize.height, 'background-color': moveBlockBackgroundColor, left: moveBlockLeft, transition: transitionLeft}">
                    <i :class="['verify-icon iconfont', iconClass]"
                       :style="{color: iconColor}"></i>
                    <div
                        class="verify-sub-block"
                        :style="{'width': blockSize.width,
                                  'height': blockSize.height,
                                  'top': '-' + (parseInt(setSize.imgHeight) - top + vSpace) + 'px',
                                  'background-image': 'url(' + defaultImg + ')',
                                  'background-size': setSize.imgWidth + ' ' + setSize.imgHeight,
                                  'background-position-y': '-' + top + 'px',
                                  'background-position-x': '-' + left + 'px'}"
                        v-show="  type === '2'&& showImage"></div>
                </div>
            </div>
        </div>
    </div>
</template>
<script type="text/babel">
  /**
   * VerifySlide
   * @description 滑块
   * */
  import {resetSize} from '../../lib/util'
  
  export default {
    name: 'VerifySlide',
    props: {
      type: {
        type: String,
        default: '1'
      },
      //弹出式pop，固定fixed
      mode: {
        type: String,
        default: 'fixed'
      },
      vOffset: {
        type: Number,
        default: 5
      },
      vSpace: {
        type: Number,
        default: 5
      },
      explain: {
        type: String,
        default: '向右滑动完成验证'
      },
      ReleaseValidation:{
        type: String,
        default: '松开验证'
      },
      ValidationSuccessful:{
        type: String,
        default: '验证成功'
      },
      ValidationFailed:{
        type: String,
        default: '验证失败'
      },
      RefreshVerificationVode:{
        type: String,
        default: '刷新验证码'
      },
      imgUrl: {
        type: String,
      },
      imgSize: {
        type: Object,
        default() {
          return {
            width: '400px',
            height: '200px'
          }
        }
      },
      blockSize: {
        type: Object,
        default() {
          return {
            width: '50px',
            height: '50px'
          }
        }
      },
      barSize: {
        type: Object,
        default() {
          return {
            width: '400px',
            height: '40px'
          }
        }
      }
    },
    data() {
      return {
        startSite: 0,
        text: '',
        finishText: '',
        setSize: {
          imgHeight: 0,
          imgWidth: 0,
          barHeight: 0,
          barWidth: 0
        },
        top: 0,
        left: 0,
        showImage: true,
        moveBlockLeft: undefined,
        leftBarWidth: undefined,
        // 移动中样式
        moveBlockBackgroundColor: undefined,
        leftBarBorderColor: '#ddd',
        iconColor: undefined,
        iconClass: 'icon-right',
        status: false,	//鼠标状态
        isEnd: false,		//是够验证完成
        showRefresh: true,
        transitionLeft: '',
        transitionWidth: '',
        defaultImg: '',
        slideData: {
          sliderDownTime: '',//鼠标按下时间
          travel: [],//鼠标移动轨迹
          sliderSucessTime: ''//验证成功的时间
        },
      }
    },
    computed: {
      barArea() {
        return this.$el.querySelector('.verify-bar-area')
      },
      resetSize() {
        return resetSize
      }
    },
    methods: {
      // 随机生成img src
      getRandomImg() {
        return `https://picsum.photos/${this.imgSize.width.slice(0, -2)}/${this.imgSize.height.slice(0, -2)}/?image=` + this.getRandomNumberByRange(0, 1084);
      },
      getRandomNumberByRange(start, end) {
        return Math.round(Math.random() * (end - start) + start)
      },
      init() {
        this.text = this.explain
        this.$nextTick(() => {
          let setSize = this.resetSize(this)	//重新设置宽度高度
          // 监听的问题
          for (let key in setSize) {
            this.$set(this.setSize, key, setSize[key])
          }
          
          this.randSet()
          
          this.$parent.$emit('ready', this)
        })
        
        var _this = this
        
        window.removeEventListener("touchmove", function (e) {
          _this.move(e);
        });
        window.removeEventListener("mousemove", function (e) {
          _this.move(e);
        });
        
        //鼠标松开
        window.removeEventListener("touchend", function () {
          _this.end();
        });
        window.removeEventListener("mouseup", function () {
          _this.end();
        });
        
        window.addEventListener("touchmove", function (e) {
          _this.move(e);
        });
        window.addEventListener("mousemove", function (e) {
          _this.move(e);
        });
        
        //鼠标松开
        window.addEventListener("touchend", function () {
          _this.end();
        });
        window.addEventListener("mouseup", function () {
          _this.end();
        });
        
      },
      
      //鼠标按下
      start: function (e) {
        this.startSite = e.x
        if (this.isEnd == false) {
          this.text = this.ReleaseValidation
          this.moveBlockBackgroundColor = '#337ab7'
          this.leftBarBorderColor = '#337AB7'
          this.iconColor = '#fff'
          e.stopPropagation();
          this.slideData.sliderDownTime = (new Date()).getTime()
          this.status = true;
          this.slideData.travel = []
        }
      },
      //鼠标移动
      move: function (e) {
        if (this.status && this.isEnd == false) {
          var x = e.x;
          let barWidth = this.barSize.width.split('px')[0] - this.blockSize.width.split('px')[0] + 20
          // if (!e.touches) {    //兼容移动端
          //     var x = e.clientX;
          // } else {     //兼容PC端
          //     var x = e.touches[0].pageX;
          // }
          var bar_area_left = this.getLeft(this.barArea);
          var move_block_left = x - bar_area_left //小方块相对于父元素的left值
          if (this.type !== '1') {		//图片滑动
            if (move_block_left >= this.barArea.offsetWidth - parseInt(parseInt(this.blockSize.width) / 2) - 2) {
              
              move_block_left = this.barArea.offsetWidth - parseInt(parseInt(this.blockSize.width) / 2) - 2;
            }
          } else {		//普通滑动
            if (move_block_left >= this.barArea.offsetWidth - parseInt(parseInt(this.barSize.height) / 2) + 3) {
              this.finishText = ''
              move_block_left = this.barArea.offsetWidth - parseInt(parseInt(this.barSize.height) / 2) + 3;
            } else {
              this.text = this.ReleaseValidation
            }
          }
          if (move_block_left <= 0) {
            move_block_left = parseInt(parseInt(this.blockSize.width) / 2);
          }
          //拖动后小方块的left值
          this.moveBlockLeft = this.selectWidth(x - this.startSite, barWidth) + "px"
          this.leftBarWidth = this.selectWidth(x - this.startSite, barWidth) + "px"
          this.getTravel(e.clientX, e.clientY)
        }
      },
      //获取移动轨迹
      getTravel(x, y) {
        this.slideData.travel.push((x + "_" + y))
      },
      
      selectWidth(x, y) {
        if (x <= 0) {
          return 0
        } else if (x >= y) {
          return y
        } else {
          return x
        }
      },
      
      //鼠标松开
      end: function () {
        var _this = this;
        //判断是否重合
        if (this.status && this.isEnd == false) {
          if (this.type !== '1') {		//图片滑动
            var vOffset = parseInt(this.vOffset)
            if (parseInt(this.left) >= (parseInt((this.moveBlockLeft || '').replace('px', '')) - vOffset) &&
              parseInt(this.left) <= (parseInt((this.moveBlockLeft || '').replace('px', '')) + vOffset)) {
              this.moveBlockBackgroundColor = '#5cb85c'
              this.leftBarBorderColor = '#5cb85c'
              this.iconColor = '#fff'
              this.iconClass = 'icon-check'
              this.showRefresh = false
              this.isEnd = true;
              this.slideData.sliderSucessTime = (new Date()).getTime()
              this.$parent.$emit('success', true, this.slideData)
              this.finishText = this.ValidationSuccessful
              setTimeout(function () {
                _this.text = ''
              }, 400);
            } else {
              this.moveBlockBackgroundColor = '#d9534f'
              this.leftBarBorderColor = '#d9534f'
              this.iconColor = '#fff'
              this.iconClass = 'icon-close'
              this.text = this.ValidationFailed
              this.finishText = this.ValidationFailed
              
              setTimeout(function () {
                _this.finishText = ''
                _this.refresh();
              }, 400);
              this.$parent.$emit('error', false)
            }
          } else {		//普通滑动
            if (parseInt((this.moveBlockLeft || '').replace('px', '')) >= (parseInt(this.setSize.barWidth) - parseInt(this.barSize.height) - parseInt(this.vOffset))) {
              this.moveBlockBackgroundColor = '#5cb85c'
              this.leftBarBorderColor = '#5cb85c'
              this.iconColor = '#fff'
              this.iconClass = 'icon-check'
              this.showRefresh = false
              this.finishText = this.ValidationSuccessful
              this.isEnd = true;
              this.slideData.sliderSucessTime = (new Date()).getTime()
              this.$parent.$emit('success', true, this.slideData)
            } else {
              this.text = this.ValidationFailed
              this.finishText = this.ValidationFailed
              this.moveBlockBackgroundColor = '#d9534f'
              this.leftBarBorderColor = '#d9534f'
              this.iconColor = '#fff'
              this.iconClass = 'icon-close'
              this.isEnd = true;
              setTimeout(function () {
                _this.finishText = ''
                _this.refresh()
                _this.isEnd = false
              }, 400);
              this.$parent.$emit('error', false)
            }
          }
          this.status = false;
        }
      },
      
      //随机出生点位
      randSet: function () {
        var rand1 = Math.floor(Math.random() * 9 + 1);
        var rand2 = Math.floor(Math.random() * 9 + 1);
        var top = rand1 * parseInt(this.setSize.imgHeight) / 15 + parseInt(this.setSize.imgHeight) * 0.1;
        var left = rand2 * parseInt(this.setSize.imgWidth) / 15 + parseInt(this.setSize.imgWidth) * 0.1;
        this.top = top
        this.left = left
      },
      
      refresh: function () {
        this.showRefresh = true
        this.defaultImg = this.getRandomImg()
        this.transitionLeft = 'left .3s'
        this.moveBlockLeft = 0
        this.leftBarWidth = undefined
        this.transitionWidth = 'width .3s'
        this.leftBarBorderColor = '#ddd'
        this.moveBlockBackgroundColor = '#fff'
        this.iconColor = '#000'
        this.iconClass = 'icon-right'
        this.text = this.RefreshVerificationVode
        this.finishText = ''
        this.randSet()
        this.isEnd = false
        setTimeout(() => {
          this.transitionWidth = ''
          this.transitionLeft = ''
          this.text = this.explain
        }, 400)
      },
      
      //获取left值
      getLeft: function (node) {
        let leftValue = 0;
        while (node) {
          leftValue += node.offsetLeft;
          node = node.offsetParent;
        }
        let finalvalue = leftValue;
        return finalvalue;
      }
    },
    watch: {
      // type变化则全面刷新
      type: {
        immediate: true,
        handler() {
          this.init()
        }
      },
      imgUrl: {
        immediate: true,
        handler(e, v) {
          this.defaultImg = e ? e : this.getRandomImg()
        }
      }
    },
    mounted() {
      // 禁止拖拽
      this.$el.onselectstart = function () {
        return false
      }
    },
    i18n: {
      messages: {
        'en-US': {},
        'zh-CN': {}
      }
    }
  }
</script>
