<template>
  <div class="goods">
    <div class="menu-wrapper"
         ref="menuWapper">
      <ul>
        <li v-for="(item,index) in goods"
            :key="index"
            class="menu-item border-1px"
            :class="{'current': currentIndex === index}"
            @click="selectMenu(index)">
          <span class="text">
            <span v-show="item.type > 0"
                  class="icon"
                  :class="classMap[item.type]"></span>
            {{ item.name }}
          </span>
        </li>
      </ul>
    </div>
    <div class="foods-wrapper"
         ref="foodsWrapper">
      <ul>
        <li v-for="(item, index) in goods"
            :key="index"
            class="food-list food-list-hook">
          <h1 class="title">{{ item.name }}</h1>
          <ul>
            <li v-for="(food, index) in item.foods"
                @click="selectFood(food, $event)"
                :key="index"
                class="food-item border-bottom-1px">
              <div class="icon">
                <img width="57"
                     height="57"
                     :src="food.icon"/>
              </div>
              <div class="content">
                <h2 class="name">{{ food.name }}</h2>
                <p class="desc">{{ food.description }}</p>
                <div class="extra">
                  <span>月售{{ food.sellCount }}份</span><span>好评率{{ food.rating }}%</span>
                </div>
                <div class="price">
                  <span class="now">￥{{ food.price }}</span>
                  <span class="old"
                        v-show="food.oldPrice">￥{{ food.oldPrice }}</span>
                </div>
                <div class="cartcontrol-wrapper">
                  <cart-control
                    @add="onAdd"
                    :food="food"></cart-control>
                </div>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>
    <shop-cart ref="shopCart"
               :select-foods="selectFoods"
               :delivery-price="seller.deliveryPrice"
               :min-price="seller.minPrice"></shop-cart>
    <food :food="selectedFood" ref="food"></food>
  </div>
</template>

<script>
  import { getGoods } from 'api'
  import BScroll from 'better-scroll'
  import ShopCart from '../../components/shop-cart/shop-cart'
  import CartControl from 'components/cart-control/cart-control'
  import Food from '../../components/food/food'

  export default {
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
      }
    },
    computed: {
      currentIndex() {
        for (let i = 0; i < this.listHeight.length; i++) {
          const height1 = this.listHeight[i]
          const height2 = this.listHeight[i + 1]
          if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
            return i
          }
        }
        return 0
      },
      selectFoods() {
        const foods = []
        this.goods.forEach((good) => {
          good.foods.forEach((food) => {
            if (food.count) {
              foods.push(food)
            }
          })
        })
        return foods
      }
    },
    created() {
      this._getGoods()
      this.classMap = ['decrease', 'discount', 'invoice', 'special', 'guarantee']
    },
    methods: {
      _getGoods() {
        getGoods().then((goods) => {
          this.goods = goods
          this.$nextTick(() => {
            this._initScroll()
            this._calculateHeight()
          })
        })
      },
      _initScroll() {
        this.menuScroll = new BScroll(this.$refs.menuWapper, {
          click: true
        })
        this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
          click: true,
          probeType: 3
        })

        this.foodsScroll.on('scroll', (pos) => {
          this.scrollY = Math.abs(Math.round(pos.y))
        })
      },
      _calculateHeight() {
        const foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook')
        let height = 0
        this.listHeight.push(height)
        for (let i = 0; i < foodList.length; i++) {
          const item = foodList[i]
          height += item.clientHeight
          this.listHeight.push(height)
        }
      },
      selectMenu(index) {
        const foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook')
        const el = foodList[index]
        this.foodsScroll.scrollToElement(el, 300)
      },
      onAdd(target) {
        this.$refs.shopCart.drop(target)
      },
      selectFood(food, event) {
        if (!event._constructed) {
          return
        }
        this.selectedFood = food
        this.$refs.food.show()
      }
    },
    components: {
      ShopCart,
      CartControl,
      Food
    }
  }
</script>

<style scoped lang="stylus">
  @import "../../common/stylus/mixin.styl"

  .goods
    display: flex;
    position: absolute;
    width: 100%;
    top: 174px;
    bottom: 46px;
    overflow: hidden;

    .menu-wrapper
      flex: 0 0 80px;
      width: 80px;
      background: #f3f5f7;

      .menu-item
        display: table;
        height: 54px;
        width: 56px;
        line-height: 14px;
        padding: 0 12px;

        &.current
          position: relative;
          z-index: 10;
          margin-top: -1px;
          background: #fff;
          font-weight: 700;
          .text
            border-none()

        .icon
          display: inline-block;
          width: 12px;
          height: 12px;
          margin-right: 2px;
          background-size: 12px;
          background-repeat: no-repeat;
          &.decrease
            bg-image('decrease_3');
          &.discount
            bg-image('discount_3');
          &.guarantee
            bg-image('guarantee_3');
          &.invoice
            bg-image('invoice_3');
          &.special
            bg-image('special_3');

        .text
          display: table-cell;
          width: 56px;
          vertical-align: middle;
          border-1px(rgba(7, 17, 27, 0.1))
          font-size: 12px;

    .foods-wrapper
      flex: 1;
      .title
        padding-left: 14px;
        height: 26px;
        line-height: 26px;
        border-left: 2px solid #d9dde1;
        font-size: 12px;
        color: rgb(147, 153, 159);
        background: #f3f5f7;

      .food-item
        display: flex;
        margin: 18px;
        padding-bottom: 18px;

        &:last-child
          border-none()
          margin-bottom: 0;

        .icon
          flex: 0 0 57px;
          margin-right: 10px;

        .content
          flex: 1;
          position: relative;

          .name
            margin: 2px 0 8px 0;
            height: 14px;
            line-height: 14px;
            font-size: 14px;
            color: rgb(7, 17, 27);

          .desc, .extra
            line-height: 12px;
            font-size: 12px;
            color: rgb(147, 153, 159);

          .desc
            margin-bottom: 8px;

          .extra
            line-height: 10px;

          .price
            font-weight: 700;
            line-height: 24px;

            .now
              margin-right: 8px;
              font-size: 14px;
              color: rgb(240, 20, 20);

            .old
              text-decoration: line-through;
              font-size: 10px;
              color: rgb(147, 153, 159);

          .cartcontrol-wrapper
            position: absolute;
            right: 0;
            bottom: -7px;
</style>
