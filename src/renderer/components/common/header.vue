<template>
  <header>
    <embed src="/static/logo.svg" style="margin-top:7px;margin-left:2vw;float:left" />
    <div class="mod_search_input">
        <input class="search_input__input" v-model="searchKeyWord" type="text" placeholder="搜索音乐、MV、歌单、用户" @keyup.enter="enter" >
    </div>
  </header>
</template>

<script>
export default {
  data () {
    return {
      searchKeyWord: '暗杠' // 搜索关键字
    }
  },
  methods: {
    enter () {
      this.$emit('enterFn', this.searchKeyWord)
    }
  }
}
</script>
<style scope>
header{
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  height: 60px;
  width: 100%;
  background-color: #C20C0C;
  border-bottom: 1px solid #a40011;
}
</style>

