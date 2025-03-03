<template>
  <transition name="move">
    <div v-show="showFlag" class="food" ref="food">
      <div class="food-content">
        <div class="image-header">
          <img :src="food.image" alt=""/>
          <div class="back" @click="hide">
            <i class="icon-arrow_lift"></i>
          </div>
        </div>
        <div class="content">
          <h1 class="title">{{food.name}}</h1>
          <div class="detail">
            <span class="sell-count">月售{{food.sellCount}}份</span><span class="rating">好评率{{food.rating}}%</span>
          </div>
          <div class="price">
            <span class="now">￥{{food.price}}</span><span v-show="food.oldPrice" class="old">￥{{food.oldPrice}}</span>
          </div>
          <div class="cartControl-wrapper">
            <cartControl :food="food" @add="addFood"></cartControl>
          </div>
          <transition name="fade">
            <div @click.stop.prevent="addFirst" class="buy" v-show="!food.count || food.count===0">
              加入购物车
            </div>
          </transition>
        </div>
        <split v-show="food.info"></split>
        <div class="info" v-show="food.info">
          <h1 class="title">商品介绍</h1>
          <p class="text">{{food.info}}</p>
        </div>
        <split></split>
        <div class="rating">
          <h1 class="title">商品评价</h1>
          <ratingSelect :selectType="selectType"
                        :onlyContent="onlyContent"
                        :desc="desc"
                        :ratings="food.ratings"
                        @select="selectRating"
                        @toggle="toggleContent"></ratingSelect>
          <div class="rating-wrapper">
            <ul v-show="food.ratings && food.ratings.length">
              <li v-show="needShow(rating.rateType,rating.text)" v-for="rating in food.ratings" class="rating-item">
                <div class="user">
                  <span class="name">{{rating.username}}</span>
                  <img :src="rating.avatar" class="avatar" width="12" height="12" alt=""/>
                </div>
                <div class="time">{{rating.rateTime | formatDate}}</div>
                <p class="text">
                  <span :class="{'icon-thumb_up':rating.rateType===0,'icon-thumb_down':rating.rateType===1}"></span>{{rating.text}}
                </p>
              </li>
            </ul>
            <div class="no-rating" v-show="!food.ratings || !food.ratings.length">
              暂无评价
            </div>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
  import Vue from 'vue';
  import BScroll from 'better-scroll';
  import {formatDate} from '../../common/js/date';
  import cartControl from '../cartControl/cartControl.vue';
  import split from '../split/split.vue';
  import ratingSelect from '../ratingSelect/ratingSelect.vue';
  /* eslint-disable no-unused-vars */
  const POSITIVE = 0;
  const NEGATIVE = 1;
  const ALL = 2;
  export default {
    props: {
      food: {
        type: Object
      }
    },
    data() {
      return {
        showFlag: false,
        selectType: ALL,
        onlyContent: true,
        desc: {
          all: '全部',
          positive: '推荐',
          negative: '吐槽'
        }
      };
    },
    methods: {
      show() {
        this.showFlag = true;
        this.selectType = ALL;
        this.onlyContent = false;
        this.$nextTick(() => { // DOM更新调用
          if (!this.scroll) {
              this.scroll = new BScroll(this.$refs.food, {
                  click: true
              });
          } else {
              this.scroll.refresh();
          }
        });
      },
      hide() {
        this.showFlag = false;
      },
      addFirst(event) {
        if (!event._constructed) {
            return;
        }
        this.$emit('add', event.target); // 主动触发 add 方法，'event.target' 为向父组件传递的数据
        Vue.set(this.food, 'count', 1);
      },
      addFood(target) {
        this.$emit('add', target);
      },
      selectRating(type) {
        this.selectType = type;
        this.$nextTick(() => {
          this.scroll.refresh();
        });
      },
      toggleContent() {
        this.onlyContent = !this.onlyContent;
        this.$nextTick(() => {
          this.scroll.refresh();
        });
      },
      needShow(type, text) {
          if (this.onlyContent && !text) {
              return false;
          }
          if (this.selectType === ALL) {
              return true;
          } else {
              return type === this.selectType;
          }
      }
    },
    filters: {
      formatDate(time) {
          let date = new Date(time);
          return formatDate(date, 'yyyy-MM-dd hh:mm');
      }
    },
    components: {
      cartControl,
      split,
      ratingSelect
    }
  };
</script>

<style lang="stylus" type="text/stylus">
  @import "../../common/stylus/mixin.styl"
  .food
    position fixed
    left 0
    top 0
    bottom 48px
    z-index 30
    width 100%
    background #fff
    transform translate3d(0,0,0)
    &.move-enter-active, &.move-leave-active
      transition all .4s linear
    &.move-enter, &.move-leave-to
      transform translate3d(100%,0,0)
    .image-header
      position relative
      width 100%
      height 0
      padding-top 100% /* padding-top 设为width的100% 使上内边距和宽度一致*/
      img
        position absolute
        top 0
        left 0
        width 100%
        height 100%
      .back
        position absolute
        top 10px
        left 0
        .icon-arrow_lift
          display block
          padding 8px
          font-size 16px
    .content
      position relative
      padding 18px
      .title
        line-height 14px
        margin-bottom 8px
        font-size 14px
        font-weight 700
        color rgb(7,17,27)
      .detail
        margin-bottom 18px
        line-height 10px
        height 10px
        font-size 0
        .sell-count, .rating
          font-size 10px
          color rgb(147,153,159)
        .sell-count
          margin-right 12px
      .price
        font-weight 700
        line-height 24px
        .now
          margin-right 8px
          font-size 14px
          color rgb(240,20,20)
        .old
          text-decoration line-through
          font-size 10px
          color rgb(147,153,159)
      .cartControl-wrapper
        position absolute
        right 12px
        bottom 12px
      .buy
        position absolute
        right 18px
        bottom 18px
        z-index 10
        height 24px
        line-height 24px
        padding 0 12px
        box-sizing border-box
        border-radius 12px
        font-size 10px
        color #fff
        background rgb(0,160,220)
        opacity 1
        &.fade-enter-active, &.fade-leave-active
          transition all 0.2s
        &.fade-enter, &.fade-leave-active
          opacity 0
          z-index -1
    .info
      padding 18px
      .title
        line-height 14px
        margin-bottom 6px
        font-size 14px
        font-weight 600
        color rgb(7,17,27)
      .text
        line-height 24px
        font-size 12px
        padding 0 8px
        color rgb(77,85,93)
    .rating
      padding-top 18px
      .title
        line-height 14px
        margin-left 18px
        font-size 14px
        font-weight 600
        color rgb(7,17,27)
      .rating-wrapper
        padding 0 18px
        .rating-item
          position relative
          padding 16px 0
          border-bottom-1px(rgba(7,17,27,.1))
          .user
            position absolute
            top 16px
            right 0
            line-height 12px
            font-size 0
            .name
              display inline-block
              margin-right 6px
              vertical-align top
              font-size 10px
              color rgb(147,153,159)
            .avatar
              border-radius 50%
          .time
            margin-bottom 6px
            line-height 12px
            font-size 10px
            color rgb(147,153,159)
          .text
            line-height 16px
            font-size 12px
            color rgb(7,17,27)
            .icon-thumb_up, .icon-thumb_down
              margin-right 4px
              line-height 16px
              font-size 12px
            .icon-thumb_up
              color rgb(0,160,220)
            .icon-thumb_down
              color rgb(147,153,159)
        .no-rating
          padding 16px 0
          font-size 12px
          color rgb(147,153,159)
</style>
