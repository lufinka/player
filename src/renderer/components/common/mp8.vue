<template>
  <footer>
    <div class="handle" ref="handle">
      <i class="prev" @click="playPrev"></i>
      <i :class="{play:isplayed == 2,stop:isplayed != 2}" @click="switchplay"></i>
      <i class="next" @click="playNext"></i>
    </div>
    <div class="info" ref="info">
      <div class="mation">
        <div class="artists">{{data.name}} / {{data.artists[0].name}}</div>
        <div class="time">{{duration | toMinutes}} / {{data.duration | toMinutes}}</div>
      </div>
      <div class="player_progress" id="progress">
          <div class="player_progress__inner" id="spanplayer_bgbar" title="[快进ctrl+alt+→][快退ctrl+alt+←]">
              <div class="player_progress__load" ref="downloadbar" style="width: 100%;"></div>
              <div class="player_progress__play" :style="{width: duration/data.duration*100+'%'}" id="spanplaybar"><i class="player_progress__dot" ref="spanprogress_op"></i></div>
          </div>
      </div>
    </div>
    <div class="voice">
      <i></i>
      <p></p>
    </div>
  </footer>
</template>

<script>
export default {
  data () {
    return {
      index: 0,
      duration: 0, // 音乐时长
      timer: null, // 定时器
      begin: 0, // 进度条点击位置
      move: 0, // 移动距离
      barWidth: 0, // 进度条宽度
      offsWidth: 0 // 进度条距离窗口左边的距离
    }
  },
  created () {
    this.index = (this.now === -1 ? 0 : this.now)
  },
  mounted () {
    this.barWidth = this.$refs.downloadbar.offsetWidth
    this.offsWidth = this.$refs.info.offsetLeft
    this.bindEvent()
  },
  props: {
    now: {
      type: Number
    },
    isplayed: {
      type: Number
    },
    data: {
      type: Object,
      default: function () {
        return {
          name: 'null',
          artists: [{name: 'null'}],
          duration: 0
        }
      }
    }
  },
  filters: {
    toMinutes: function (num) {
      const toDouble = function (n) {
        return n > 9 ? n : '0' + n
      }
      return toDouble(parseInt(num / 1000 / 60)) + ':' + toDouble(parseInt(num / 1000 % 60))
    }
  },
  methods: {
    indexChange: function (newVal) {
      this.duration = 0
      clearInterval(this.timer)
      if (this.isplayed !== 2) {
        this.timer = setInterval(() => {
          this.duration += 1000
        }, 1000)
      }
    },
    switchplay () {
      this.$emit('swit')
      if (this.isplayed === 1) {
        clearInterval(this.timer)
      } else {
        this.timer = setInterval(() => {
          this.duration += 1000
        }, 1000)
      }
    },
    playPrev () {
      if (this.index === 0) {
        this.index = 0
      } else {
        clearInterval(this.timer)
        this.index = this.index - 1
      }
      this.$emit('playFn', this.index)
    },
    playNext () {
      clearInterval(this.timer)
      this.index = this.index + 1
      this.$emit('playFn', this.index)
    },
    bindEvent (e) {
      let me = this
      this.$refs.spanprogress_op.onmousedown = function (e) {
        me.begin = e.target.offsetLeft
        let nowdur = me.duration
        document.onmousemove = function (e) {
          clearInterval(me.timer)
          me.move = e.pageX
          if (me.move <= me.offsWidth) {
            me.duration = 0
          } else if (me.move >= (me.barWidth + me.offsWidth)) {
            me.duration = me.data.duration
          } else {
            me.duration = nowdur + ((me.move - me.begin - me.offsWidth) * me.data.duration / me.barWidth)
          }
        }
        document.onmouseup = function (e) {
          document.onmousemove = null
          me.$emit('prog', me.duration)
          me.timer = setInterval(() => {
            me.duration += 1000
          }, 1000)
          document.onmouseup = null
        }
      }
    }
  }
}
</script>
<style scope>
footer{
  border-top: 1px solid rgba(225,225,225,.3);
  padding-top: 20px;
  position: fixed;
  display: flex;
  align-items: center;
  justify-content: center;
  bottom: 10px;
  left: 0;
  right: 0;
  height: 90px;
  color: #ffffff;
  width: 100%;
}
.play{
  cursor: pointer;
  width: 32px;
  margin: 0 50px;
  height: 32px;
  display: block;
  float: left;
  background: url(/static/play1.png);
}
.stop{
  cursor: pointer;
  width: 32px;
  height: 32px;
  margin: 0 50px;
  display: block;
  float: left;
  background: url(/static/pause.png);
}

.prev{
  cursor: pointer;
  width: 32px;
  height: 32px;
  display: block;
  float: left;
  background: url(/static/prev.png);
}

.next{
  cursor: pointer;
  width: 32px;
  height: 32px;
  display: block;
  float: left;
  background: url(/static/next.png);
}

.info{
    margin-left: 30px;
    width: 60vw;
    height: 32px;
}
.mation{
  width: 100%;
  height: 24px;
  line-height: 20px;
  overflow: hidden;
  font-size: 14px;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
  color: rgba(225,225,225,.8);
}
.artists:hover,time:hover{
    color: #ffffff;
}
.artists{
  float: left;
}
.time{
  float: right;
  font-size: 16px;
}
.player_progress {
    padding-top: 6px;
    height: 8px;
    cursor: pointer
}

.player_progress__inner {
    position: relative;
    height: 2px;
    filter: progid:DXImageTransform.Microsoft.gradient(enabled='true', startColorstr='#19FFFFFF', endColorstr='#19FFFFFF');
    background: rgba(255,255,255,.1)
}

.player_progress__load {
    height: 2px;
    filter: progid:DXImageTransform.Microsoft.gradient(enabled='true', startColorstr='#33FFFFFF', endColorstr='#33FFFFFF');
    background: rgba(255,255,255,.2)
}

.player_progress__play {
    position: absolute;
    top: 0;
    left: 0;
    height: 2px;
    filter: progid:DXImageTransform.Microsoft.gradient(enabled='true', startColorstr='#B2FFFFFF', endColorstr='#B2FFFFFF');
    background: rgba(255,255,255,.7)
}

.player_progress__dot {
    position: absolute;
    background-color: #fff;
    border-radius: 5px;
    top: -4px;
    right: -4px;
    width: 10px;
    height: 10px;
    background-position: 0 -170px;
    opacity: 1;
    filter: none
}
</style>

