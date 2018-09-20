<template>
<div class="player-box" :style="{height: `${height}px`}">
  <div class="palyer-cover">
    <mt-swipe :auto="3000" :showIndicators="false">
      <mt-swipe-item v-for="(item, i) in swipper" :key="i">
        <img :src="item" alt="封面图">
      </mt-swipe-item>
    </mt-swipe>
  </div>
  <div class="palyer-music clearfix">
    <!-- 播放按钮 -->
    <div class="play-btn" @click.stop="handleStart">
      <img src="../../assets/images/desktop_lyrics_pause@2x.png" alt="" v-if="start">
      <img src="../../assets/images/desktop_lyrics_play@2x.png" alt="" v-else>
    </div>
    <!-- 进度条 -->
    <div class="progress-box" ref="progress" @click="handleProgress($event)">
      <div class="progress-bg"></div>
      <div class="progress-bar" ref="percent" :style="{width:`${Number(progress*100).toFixed(2) >=100 ? 100 : Number(progress*100).toFixed(2)}%`}">
          <span class="progress-lot" ref="inner" @mousedown.prevent.stop="handleBarDown($event)" @touchstart.prevent.stop="handleBarDown($event)"></span>
        </div>
    </div>
    <!-- 时间 -->
    <div class="palyer-time">
      {{`0${currentTime >=60 ? parseInt(currentTime/60) : '0'}:${(currentTime%60 >= 10 ?  currentTime%60 : `0${currentTime%60}`)}`}}/{{`0${parseInt(longTime/60)}:${longTime%60}`}}
    </div>
    <audio id="player" preload>
      <source :src="mv.mp3">
    </audio>
  </div>
</div>
</template>
<script>
import { Swipe, SwipeItem } from 'mint-ui';
import banner1 from '@/assets/images/1.jpg'
import banner2 from '@/assets/images/2.jpg'
import banner3 from '@/assets/images/3.jpg'
import banner4 from '@/assets/images/4.jpg'
import banner5 from '@/assets/images/5.jpg'
import mv from '@/assets/mp3/mp.mp3'
export default {
  data () {
    return {
      start: false, //定义播放按钮是显示播放还是暂停  false表示暂停 true表示播放
      longTime: 0, //定义音频的总时长
      currentTime: 0, //当前播放时间
      progress: 0, //进度条宽度
      height: 2 * document.body.clientWidth - document.body.offsetWidth,
      move: {
        status: false, // 是否可拖动
        startX: 0, // 记录最开始点击的X坐标
        left: 0, // 记录当前已经移动的距离
      },
      mv: mv,
      swipper: [
        banner1,
        banner2,
        banner3,
        banner4,
        banner5
      ]
    }
  },
  components: {
    [Swipe.name]: Swipe,
    [SwipeItem.name]: SwipeItem
  },
  mounted () {
    let _self = this
    window.onresize = () => {
      this.height = 2 * document.body.clientWidth - document.body.offsetWidth
    }
    this.bindEvents()
    let player = document.querySelector('#player')
    player.load()
    // 播放器加载完成 没有播放时获取音频的时长
    player.oncanplay = function () {
      _self.longTime = parseInt(this.duration)
    }
    // 监听播放进度
    player.addEventListener('timeupdate', function () {
      _self.currentTime = parseInt(this.currentTime)
      _self.progress = _self.longTime === 0 ? 0 : this.currentTime / _self.longTime
    })
  },
  methods: {
    //添加绑定事件
    bindEvents () {
      document.addEventListener('mousemove', this.handleBarMove);
      document.addEventListener('mouseup', this.handleBarUp);
      document.addEventListener('touchmove', this.handleBarMove);
      document.addEventListener('touchend', this.handleBarUp)
    },
    // 开始 暂停播放
    handleStart () {
      let player = document.querySelector('#player')
      this.start = !this.start
      this.start ? player.play() : player.pause()
    },
    // 点击进度条事件
    handleProgress (e) {
      let player = document.querySelector('#player')
      let rect = this.$refs.progress.getBoundingClientRect()
      let progressWidth = this.$refs.progress.clientWidth
      let clickX = e.clientX - rect.left
      this.progress = clickX / progressWidth
      this.start = true
      player.currentTime = this.progress * this.longTime
      player.play()
    },
    // 鼠标按下事件
    handleBarDown (e) {
      this.move.status = true;
      this.move.startX = e.clientX || e.touches[0].pageX;
      this.move.left = this.$refs.inner.clientWidth
    },
    //鼠标/触摸移动事件
    handleBarMove (e) {
      // console.log(e)
      if (!this.move.status) {
        return false
      } else {
        // 获取dom的位置信息
        let rect = this.$refs.progress.getBoundingClientRect().left
        // 获取播放器dom对象
        let player = document.querySelector('#player')
        // 触摸移动结束事件 计算进度值
        let endX = e.clientX || e.touches[0].pageX;
        let progressWidth = this.$refs.progress.clientWidth
        this.progress = (endX - rect) / progressWidth
        player.currentTime = this.progress * this.longTime
        player.play()
        this.start = true
      }
    },
    //鼠标/触摸释放事件
    handleBarUp (e) {
      e.stopPropagation();
      if (this.move.status) {
        this.move.status = false;
        // 获取dom的位置信息
        return false
      }
    },
  }
}
</script>
<style lang="less" scoped>
.player-box{
  position: relative;
  .palyer-cover{
    position: absolute;
    top: 0;
    left: 0;
    height: 100%;
    width: 100%;
    background: #F3F3F3;
    .mint-swipe-item{
      background: #F3F3F3;
      img{
        width: 100%;
      }
    }
  }
  .palyer-music{
    width: 100%;
    height: 52px;
    position: absolute;
    bottom: 0;
    left: 0;
    padding: 0 10px;
    box-sizing: border-box;
    display: flex;
    justify-content: center;
    align-items: center;
    background:linear-gradient(360deg,rgba(0,0,0,0.5),rgba(0,0,0,0));
    .play-btn{
      height: 100%;
      width: 32px;
      display: flex;
      align-items: center;
      img{
        width: 32px;
      }
    }
    .progress-box{
      flex: 1;
      margin: 0 10px;
      display: flex;
      position: relative;
      .progress-bg{
        width: 100%;
        height: 2px;
        position: absolute;
        top: 0;
        left: 0;
        background:rgba(255,255,255,1);
        border-radius:4px;
      }
      .progress-bar{
        width: 20%;
        height: 2px;
        background: #E91E63;
        position: absolute;
        .progress-lot{
          display: block;
          width: 19px;
          height: 19px;
          background: url('../../assets/images/musicplayer_icon_volume_off@2x.png') no-repeat center;
          background-size: 100%;
          position: absolute;
          top: -9.5px;
          right: -9.5px;
        }
      }
    }
    .palyer-time{
      font-size: 12px;
      color: #fff;
    }
  }
}
</style>
