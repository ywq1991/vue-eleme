<template>
  <div class="goods">
    <div class="menu-wrapper" ref="menuWrapper">
      <!--vue1.0 使用v-el/v-ref  vue2.0使用 ref-->
      <ul>
        <li v-for="(item,index) in goods" class="menu-item" :class="{'current' : currentIndex === index}"
        @click="selectMenu(index,$event)">
          <span class="text">
            <span v-show="item.type > 0" class="icon" :class="classMap[item.type]"></span>
            {{item.name}}
          </span>
        </li>
      </ul>
    </div>
    <div class="foods-wrapper" ref="foodsWrapper">
      <!--vue1.0 使用v-el/v-ref  vue2.0使用 ref-->
      <ul>
        <li v-for="item in goods" class="food-list food-list-hook">
          <h1 class="title">{{item.name}}</h1>
          <ul>
            <li @click="selectFood(food, $event)" v-for="food in item.foods" class="food-item">
              <div class="icon">
                <img :src="food.icon" width="57" height="57" alt=""/>
              </div>
              <div class="content">
                <h2 class="name">{{food.name}}</h2>
                <p class="desc">{{food.description}}</p>
                <div class="extra">
                  <span class="count">月售{{food.sellCount}}份</span><span>好评率{{food.rating}}%</span>
                </div>
                <div class="price">
                  <span class="now">￥{{food.price}}</span>
                  <span v-show="food.oldPrice" class="old">￥{{food.oldPrice}}</span>
                </div>
                <div class="cartControl-wrapper">
                  <!--监听子组件触发的 add 事件,然后调用 drop 方法-->
                  <cartControl :food="food" @add="drop"></cartControl>
                </div>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>
    <shopCart ref="shopcart"
              :select-foods="selectFoods"
              :delivery-price="seller.deliveryPrice"
              :min-price="seller.minPrice"></shopCart>

    <food :food="selectedFood" ref="food" @add="drop"></food>
  </div>
</template>

<script>
  import BScroll from 'better-scroll';
  import shopCart from '../shopCart/shopCart.vue';
  import cartControl from '../cartControl/cartControl.vue';
  import food from '../food/food.vue';
  const ERR_OK = 0;
  export default{
     props: {
         seller: {
             type: Object
         }
     },
    data() {
         return {
             goods: [],
             listHeight: [],
             scrollY: 0,
             selectedFood: {}
         };
    },
    computed: {
     currentIndex() {
        for (let i = 0, l = this.listHeight.length; i < l; i++) {
            let height1 = this.listHeight[i];
            let height2 = this.listHeight[i + 1];
            if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
                return i;
            }
        }
      },
      selectFoods() {
        let foods = [];
        this.goods.forEach((good) => {
          good.foods.forEach((food) => {
              if (food.count) {
                  foods.push(food);
              }
          });
        });
        return foods;
      }
    },
    created() {
       this.$http.get('/api/goods').then((response) => {
           response = response.body;
           if (response.errno === ERR_OK) {
               this.goods = response.data;
               this.$nextTick(() => { // DOM发生变化之后调用
                 this._initScroll();
                 this._calculateHeight();
               });
           }
       });
      this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee '];
    },
    methods: {
       drop(target) {
         // 体验优化，异步执行下落动画
           this.$nextTick(() => {
               this.$refs.shopcart.drop(target);
           });
       },
       selectMenu(index, event) {
         if (!event._constructed) {
             return;
         }
         let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
         let el = foodList[index];
         this.foodsScroll.scrollToElement(el, 300);
       },
       selectFood(food, event) {
        if (!event._constructed) {
          return;
        }
        this.selectedFood = food;
        this.$refs.food.show();
       },
       _initScroll() {
         // this.$refs.menuWrapper 取dom元素
         this.menuScroll = new BScroll(this.$refs.menuWrapper, {
           click: true
         });
         this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
           click: true,
           probeType: 3
         });
         this.foodsScroll.on('scroll', (pos) => {
             this.scrollY = Math.abs(Math.round(pos.y));
         });
       },
      _calculateHeight() {
            let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
            let height = 0;
            this.listHeight.push(height);
            for (let i = 0, l = foodList.length; i < l; i++) {
                let item = foodList[i];
                height += item.clientHeight;
                this.listHeight.push(height);
            }
       }
    },
    components: {
      shopCart,
      cartControl,
      food
    }
  };
</script>

<style lang="stylus" type="text/stylus">
  @import "../../common/stylus/mixin.styl"
  .goods
    display flex
    position absolute
    top 174px
    bottom 46px
    width 100%
    overflow hidden
    .menu-wrapper
      flex 0 0 80px
      width 80px
      background #f3f5f7
      .menu-item
        display table
        height 54px
        width 56px
        padding 0 12px
        line-height 14px
        &.current
          position relative
          margin-top -1px
          z-index 10
          background #fff
          font-weight 700
          .text
            border-none()
      .icon
        display inline-block
        vertical-align top
        width 12px
        height 12px
        margin-right 2px
        background-size 12px 12px
        background-repeat  no-repeat
        &.decrease
          bg-image('decrease_3')
        &.discount
          bg-image('discount_3')
        &.guarantee
          bg-image('guarantee_3')
        &.invoice
          bg-image('invoice_3')
        &.special
          bg-image('special_3')
      .text
        display table-cell
        width 56px
        vertical-align middle
        font-size 12px
        border-bottom-1px(rgba(7,17,27,.1))
    .foods-wrapper
      flex 1
      .title
        padding-left 14px
        height 26px
        line-height 26px
        border-left 2px solid #d9dde1
        font-size 12px
        color rgb(147,153,159)
        background #f3f5f7
      .food-item
        display flex
        margin 18px
        padding-bottom 18px
        border-bottom-1px(rgba(7,17,27,.1))
        &:last-child
          margin-bottom 0
          border-none()
        .icon
          flex 0 0 57px
          margin-right 10px
        .content
          flex 1
          .name
            margin 2px 0 8px 0
            height 14px
            line-height 14px
            font-size 14px
            color rgb(7,17,27)
          .desc, .extra
            line-height 10px
            font-size 10px
            color rgb(147,153,159)
          .desc
            line-height 12px
            margin-bottom 8px
          .extra
            .count
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
            right 0
            bottom 12px
</style>
