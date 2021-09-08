<template>
  <div id="app">
    <p> 分组颜色 </p>
    <calender :mark-list="markList" color-group />
    <br/>
    <p> 不分组颜色 </p>
    <calender :mark-list="markList.map(v => v.date)" />

    <div style="text-align: left">
      <h4>参考资料</h4>
      <ul>
        <li>
          <a href="http://www.dodobook.net/jquery/3091" target="view_window">
            使用JS生成随机的Web安全色
          </a>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import Calender from './components/Calender/Calender.vue'

export default {
  name: 'App',
  components: {
    Calender
  },
  data() {
    return {
      markList: [
        {
          date: '2021-9-9',
          group: 1
        },
        {
          date: '2021-9-10',
          group: 1
        },
        {
          date: '2021-9-12',
          group: 2
        },
        {
          date: '2021-9-13',
          group: 3
        }
      ]
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  width: 400px;
}
</style>
