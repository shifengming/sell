<template>
  <div>
    <!--头部-->
    <v-header :seller="seller"></v-header>
    <!--标签-->
    <div class="tab border-1px">
      <div class="tab-item">
        <router-link to="/goods">商品</router-link>
      </div>
      <div class="tab-item">
        <router-link to="/ratings">评论</router-link>
      </div>
      <div class="tab-item">
        <router-link to="/seller">商家</router-link>
      </div>
    </div>
    <!--内容-->
    <keep-alive>  <!-- 保存状态 -->
      <router-view :seller="seller"></router-view>
    </keep-alive>

  </div>
</template>

<script>
  import {urlParse} from './common/js/util'
  import header from './components/header/header';
  const ERR_OK = 0;
  export default {
    data() {
      return {
        seller:{}
        // seller: {
        //   id: (() => {
        //     let queryParam = urlParse()
        //     console.log(queryParam);
        //     return queryParam.id
        //   })()
        // }
      }
    },
    components: {
      'v-header': header
    },
    mounted() {
      this.$axios.get('/api/seller').then((res) => {
      console.log(11);
      // this.$ajax.get('/seller').then((res) => {
          console.log(res);
        if (res.data.errno === ERR_OK) {
          this.seller = res.data.data;
        }
      });
    }
  };


</script>

<style lang="stylus" ref="stylesheet/stylus">
  @import "./common/stylus/mixin.styl"
  .tab
    display: flex
    width: 100%
    height: 40px
    line-height: 40px
    //border-bottom: 1px solid rgba(7,17,27,0.1)
    border-1px(rgba(7,17,27,0.1))
    .tab-item
      flex: 1
      text-align: center
      & > a
        display: block
        font-size: 14px
        color: rgb(77, 85, 93)
        &.active
          color: rgb(240, 20, 20)

</style>
