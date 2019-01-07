# vue-awesome-swiper

参考[vue-awesome-swiper的使用以及API整理][1]，[surmon-china/vue-awesome-swiper][2]

[1]:https://segmentfault.com/a/1190000014609379
[2]:https://github.com/surmon-china/vue-awesome-swiper

基于 Swiper4、适用于 Vue 的轮播组件，支持服务端渲染和单页应用。
如果需要回退到 Swiper3，请使用 v2.6.7 版本。

## install

~~~ xml
npm install vue-awesome-swiper@2.6.7 —save
~~~

## 引用

### mount with global(一般在程序入口文件中引入)

~~~ javascript
import Vue from 'vue'
import VueAwesomeSwiper from 'vue-awesome-swiper'

// require styles
import 'swiper/dist/css/swiper.css'

Vue.use(VueAwesomeSwiper, /* { default global options } */)
~~~

### mount with component

~~~ javascript
import ’swiper/dist/css/swiper.css’
import {swiper, swiperSlide} from 'vue-awesome-swiper’
export default {
  component: {
    swiper: swiper,
    swiperSlide: swiperSlide
  }
}
~~~

## 用法示例

### SPA

~~~ javascript
<!-- The ref attr used to find the swiper instance -->
<template>
  <div class="wraper">
    <swiper :options="swiperOption" ref="mySwiper" @someSwiperEvent="callback">
      <!-- slides -->
      <swiper-slide>I'm Slide 1</swiper-slide>
      <swiper-slide>I'm Slide 2</swiper-slide>
      <swiper-slide>I'm Slide 3</swiper-slide>
      <swiper-slide>I'm Slide 4</swiper-slide>
      <!-- Optional controls -->
      <div class="swiper-pagination"  slot="pagination"></div>
      <div class="swiper-button-prev" slot="button-prev"></div>
      <div class="swiper-button-next" slot="button-next"></div>
      <div class="swiper-scrollbar"   slot="scrollbar"></div>
    </swiper>
  </div>
</template>

<script>
  export default {
    name: 'carrousel',
    data() {
      return {
        swiperOption: {
          // some swiper options/callbacks
          // 所有的参数同 swiper 官方 api 参数
          // ...
          pagination: '.swiper-pagination',
        }
      }
    },
    computed: {
      swiper() {
        return this.$refs.mySwiper.swiper
      }
    },
    mounted() {
      // current swiper instance
      // 然后你就可以使用当前上下文内的swiper对象去做你想做的事了
      console.log('this is current swiper instance object', this.swiper)
      this.swiper.slideTo(3, 1000, false)
    }
  }
</script>
~~~

### Async data example

~~~ html
<!-- The ref attr used to find the swiper instance -->
<template>
  <div class="wrapper">
    <swiper :options="swiperOption" v-if="showSwiper">
      <swiper-slide v-for="item of swiperList" v-bind:key="item.id">
        <img class="swiper-img" v-bind:src="item.imgUrl"/>
      </swiper-slide>
      <div class="swiper-pagination" slot="pagination"></div>
    </swiper>
  </div>
</template>

<script>
  export default {
    name: 'HomeSwiper',
    props: {
      swiperList: Array
    },
    data: function () {
      return {
        swiperOption: {
          // some swiper options/callbacks
          // 所有的参数同 swiper 官方 api 参数
          pagination: '.swiper-pagination',
          // 支持循环轮播
          loop: true
        }
      }
    },
    computed: {
      showSwiper: function () {
        return this.swiperList.length
      }
    }
  }
</script>

<style lang="stylus" scoped>
  // 样式穿透：将wrapper节点下面所有节点或者组件中的节点使用到的相应样式进行修改
  .wrapper >>> .swiper-pagination-bullet-active {
    background: #ffffff
  }
  .wrapper {
    // 前四行为宽高比自适应所需代码
    overflow: hidden;
    width: 100%;
    height: 0;
    padding-bottom: 31.25%;
    background: #eeeeee
  }
  .swiper-img {
    width: 100%
  }

</style>
~~~
