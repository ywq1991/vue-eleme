<template>
  <div id="app">
    <v-header :seller="seller"></v-header>
    <div class="tab border-1px">
     <div class="tab-item">
       <router-link :to="{path:'/goods'}">商品</router-link>
     </div>
     <div class="tab-item">
       <router-link :to="{path:'/ratings'}">评论</router-link>
     </div>
     <div class="tab-item">
       <router-link :to="{path:'/seller'}">商家</router-link>
     </div>
    </div>
    <keep-alive>
      <router-view :seller="seller"></router-view>
    </keep-alive>
  </div>
</template>

<script>
  import {urlParse} from './common/js/util';
  import header from './components/header/header.vue';
  const ERR_OK = 0;
  export default {
    data () {
      return {
        seller: {
            id: (() => {
                let queryParam = urlParse();
                return queryParam.id;
            })()
        }
      };
    },
    created () {
      this.$http.get('/api/seller?id=' + this.seller.id).then((response) => {
        response = response.body;
        if (response.errno === ERR_OK) {
          this.seller = Object.assign({}, this.seller, response.data);
        }
      });
    },
    components: {
      'v-header': header
    }
  };
</script>

<style lang="stylus" type="text/stylus">
  @import "./common/stylus/mixin.styl"
  #app
    .tab
      display flex
      width 100%
      height 40px
      line-height 40px
      border-bottom-1px(rgba(7,17,27,0.1))
      .tab-item
        flex 1
        text-align center
        &>a
          display block
          font-size 14px
          color rgb(77,85,93)
          &.active
            color rgb(240,20,20)
</style>
