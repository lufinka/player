<template>
  <div id="wrapper">
    <div class="bg"></div>
    <Header @enterFn="submit"></Header>
    <main>
        <ul class="song-list">
          <li v-for="(item,index) in songList" :key="index" @click="broadcast(index)">
            <p><img :src="item.artists[0].img1v1Url" style="width:25px;height:25px;" alt="">{{item.name}}</p> <p>{{item.artists[0].name}}</p> <p>{{item.duration | toMinutes}}</p>
          </li>
        </ul>
        <audio :src="songUrl" ref="audio" controls="controls" style="display:none"></audio>
        <!-- <system-information></system-information> -->
    </main>
    <mp8 :now="index" ref="mp8" :data="songList[index]" :isplayed.sync="played" @playFn="broadcast" @swit="switchplay" @prog="progress"></mp8>
  </div>
</template>

<script>
import SystemInformation from './LandingPage/SystemInformation'
import header from './common/header'
import mp8 from './common/mp8'
import api from '../server/api'
// import { setInterval } from 'timers'
export default {
  data () {
    return {
      songUrl: '', // 播放ing歌曲链接
      songList: [], // 歌曲列表
      index: -1, // 当前播放音乐的index
      played: 2, // 是否需要播放音乐
      mode: 2 // 1顺序播放 2 随机播放 3单曲循环
    }
  },
  watch: {
    played: function (newVal) {
      newVal === 1 ? this.$refs.audio.play() : this.$refs.audio.pause()
    },
    index: function (newVal) {
      this.$refs.mp8.indexChange()
    }
  },
  name: 'landing-page',
  components: {'SystemInformation': SystemInformation, 'Header': header, 'mp8': mp8},
  filters: {
    toMinutes: function (num) {
      const toDouble = function (n) {
        return n > 9 ? n : '0' + n
      }
      return toDouble(parseInt(num / 1000 / 60)) + ':' + toDouble(parseInt(num / 1000 % 60))
    }
  },
  methods: {
    open (link) {
      this.$electron.shell.openExternal(link)
    },
    async submit (kw) {
      this.songList = []
      this.index = 0
      await (this.played = 2)
      this.$refs.mp8.indexChange()
      let res = await this.$http.get(api.search, {params: {'keywords': kw}})
      this.songList.push(...res.data.result.songs)
      this.getUrl(0)
    },
    async broadcast (index) {
      event.stopPropagation()
      var me = this
      me.played = 4
      if (this.index === index) {
        this.played = 1
      } else {
        this.index = index
        await this.getUrl(this.index)
        me.$refs.audio.onloadstart = (
          me.played = 1
        )
        me.$refs.audio.onended = (
          this.$refs.mp8.indexChange()
        )
      }
      // setInterval(function () {
      //   console.log(me.$refs.audio.duration, me.$refs.audio.currentTime)
      // }, 1000)
      if (this.mode === 1) {
        index++
      } else if (this.mode === 2) {
        index = parseInt(Math.random() * this.songList.length)
      }
      this.$refs.audio.onended = function () {
        me.broadcast(index)
      }
    },
    progress (num) {
      this.$refs.audio.currentTime = num / 1000
    },
    async getUrl (num) {
      let res = await this.$http.get(api.url, {params: {'id': this.songList[num].id}})
      this.songUrl = res.data.data[0].url
    },
    switchplay () {
      this.played = (this.played === 1 ? 2 : 1)
    }
  }
}
</script>

<style>
@import url("https://fonts.googleapis.com/css?family=Source+Sans+Pro");

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family: "Helvetica Neue",Helvetica,"PingFang SC","Hiragino Sans GB","Microsoft YaHei","微软雅黑",Arial,sans-serif;  
  -webkit-touch-callout: none;
  -webkit-user-select: none;
  -khtml-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}
.search_input__input{
  border: 1px solid #c9c9c9;
  line-height: 36px;
  height: 36px;
  float: right;
  width: 40vw;
  margin-right: 4vw;
  text-indent: 5px;
  margin-top: 12px;
  font-size: 16px;
  position: relative;
  border-radius: 3px;
  background-color: #fff;
  -webkit-transition: width .6s ease-out;
  transition: width .6s ease-out;
}
body {
  font-family: "Source Sans Pro", sans-serif;
  background-color: rgb(75, 44, 20);
  padding-top: 70px;
}
.bg {
  height: 100vh;
  background-image: url(../assets/logo.png);
  position: fixed;
  left: 0;
  top: 0;
  z-index: -1;
  width: 100vw;
  padding-top: 50px;
  -webkit-filter: blur(65px);
  background-repeat: no-repeat;
  background-size: cover;
  filter: blur(65px);
  opacity: .6;
  -webkit-transform: translateZ(0);
  transform: translateZ(0);
}
.song-list{
  padding: 4vh 4vw;
  max-height: 70vh;
  overflow-y:scroll;
}
.song-list::-webkit-scrollbar { /*滚动条整体样式*/
    width: 5px; /*高宽分别对应横竖滚动条的尺寸*/
    height: 1px;
}

.song-list::-webkit-scrollbar-thumb { /*滚动条里面小方块*/
    border-radius: 10px;
    -webkit-box-shadow: inset 0 0 5px rgba(228, 57, 60, 0.2);
    background: rgba(20, 20, 50, 0.6);
    position: absolute;
}

.song-list::-webkit-scrollbar-track { /*滚动条里面轨道*/
    -webkit-box-shadow: inset 0 0 5px rgba(228, 57, 60, 0.2);
    border-radius: 10px;
    background: transparent;
    position: absolute;
}
.song-list ul{
  overflow: hidden;
}
.song-list li{
  display: flex;
  align-items: center;
  justify-content: space-between;
  list-style: none;
  height: 32px;
  color: #fff;
  line-height: 32px;
  float: left;
  width: 100%;
  padding-bottom: 1vh;
}

.song-list li p{
  flex: 1
}
.song-list li p:first-child{
  flex: 3;
  overflow: hidden;
  font-size: 14px;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}
.song-list li p:last-child{
  text-align: right;
}
.song-list li img{
  float: left;
  margin-right: 1vw;
}
</style>
