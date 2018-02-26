<template>
  <div id="app">
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'app'
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
.clearfix:after, .clearfix:before {
  display: table;
  content: " "
}
.clearfix:after {
  clear: both
}
#app {
  font-family : 'MicrosoftYaHei','微软雅黑';
  margin : 0;
  padding: 0;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  /*overflow-y: auto;*/
}
#app ::-webkit-scrollbar {width: 5px;border-radius: 5px; }
#app ::-webkit-scrollbar-track {background-color: #cccccc;border-radius: 5px; } /* 滚动条的滑轨背景颜色 */
#app ::-webkit-scrollbar-thumb {background-color: rgba(0, 0, 0, 0.2); } /* 滑块颜色 */
#app ::-webkit-scrollbar-button {background-color: #ffffff;border-radius: 5px; } /* 滑轨两头的监听按钮颜色 */
#app ::-webkit-scrollbar-corner {background-color: black;border-radius: 5px; } /* 横向滚动条和纵向滚动条相交处尖角的颜色 */
</style>
