<template>
  <div id="app">
    <nav class="navbar navbar-inverse navbar-fixed-top">
      <div class="container-fluid">
        <div class="navbar-header">
          <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#navbar" aria-expanded="false" aria-controls="navbar">
            <span class="sr-only">Toggle navigation</span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
          </button>
          <a class="navbar-brand">我家附近的地图</a>
        </div>
        <div id="navbar" class="navbar-collapse collapse">
          <form class="navbar-form navbar-right">
            <div class="input-group">
              <span class="input-group-btn">
                <button class="btn btn-primary" type="button">Filter</button>
              </span>
              <input type="text" class="form-control" placeholder="地点，场所..." v-model="keyword">
            </div>
          </form>
          <ul class="nav navbar-nav navbar-right navbartop" v-for="item of list">
            <li>
              <a href="#" data-toggle="collapse" data-target="#navbar" aria-expanded="false" aria-controls="navbar" @click="showInfo(item)">{{ item.title }}</a>
            </li>
          </ul>
        </div>
      </div>
    </nav>

    <div class="container-fluid">
      <div class="row">
        <div class="col-sm-3 col-md-2 sidebar">
          <ul class="nav nav-sidebar" id="nav-sidebar">
            <li v-for="item of list">
              <a href="#" @click="showInfo(item)">{{ item.title }}</a>
            </li>
          </ul>
        </div>
        <b-map :list="list"></b-map>
      </div>
    </div>

  </div>
</template>

<script>
import bMap from './components/bMap'
export default {
  name: 'app',
  data(){
    return {
      locations:[],
      list:[],
      keyword:""
  }},
  methods:{
    showInfo(place) {    //触发百度EventWrapper绑定的marker的点击事件
        var mkr = place.marker;
        BMapLib.EventWrapper.trigger(mkr, 'click', {
            'type': 'onclick',
            target: mkr
        });
    },
    filterHandle() {    //过滤方法
      var k =this.keyword;
      this.list = k != "" ? this.list = this.locations.filter(function (item) {
        return item.title.indexOf(k) !== -1;
      }) : this.locations;
    }
  },
  watch:{
    keyword() {
      this.filterHandle()
    }
  },
  components: {
    bMap
  },
  created () {   /* 这个是vue的钩子函数，当new Vue()实例创建完毕后执行的函数 */
      this.$http.get('/api/locations').then((data) => {   /* 调用vue的ajax来请求数据，promise语法，并用es6的箭头函数 */
      this.list = data.body.data;
      this.locations = data.body.data;
      })
  }
}
</script>

<style src="./assets/css/style.css"></style>
