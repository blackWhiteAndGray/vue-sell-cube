<template>
  <div class="seller" ref="seller">
    <div class="seller-content">
      <div class="overview">
        <h1 class="title">{{ seller.name }}</h1>
        <div class="desc border-bottom-1px">
          <star :size="36" :score="seller.score"></star>
          <span class="text">({{ seller.ratingCount }})</span>
          <span class="text">月售{{ seller.sellCount }}单</span>
        </div>
        <ul class="remark">
          <li class="block">
            <h2>起送价</h2>
            <div class="content">
              <span class="stress">{{ seller.minPrice }}</span>
            </div>
          </li>
          <li class="block">
            <h2>商家配送</h2>
            <div class="content">
              <span class="stress">{{ seller.deliveryPrice }}</span>
            </div>
          </li>
          <li class="block">
            <h2>平均配送时间</h2>
            <div class="content">
              <span class="stress">{{ seller.deliveryTime }}分钟</span>
            </div>
          </li>
        </ul>
        <div class="favorite" @click="toggleFavorite">
          <span class="icon-favorite" :class="{'active': favorite}"></span>
          <span class="text">{{ favoriteText }}</span>
        </div>
      </div>
      <split></split>
      <div class="bulletin">
        <h1 class="title">公共与活动</h1>
        <div class="content-wrapper border-bottom-1px">
          <p class="content">{{ seller.bulletin }}</p>
        </div>
        <ul v-if="seller.supports" class="supports">
          <li class="support-item border-bottom-1px" v-for="(item, index) in seller.supports" :key="index">
            <span class="icon" :class="classMap[item.type]"></span>
            <span class="text">{{ item.description}}</span>
          </li>
        </ul>
      </div>
      <split></split>
      <div class="pics">
        <h1 class="title">商家实景</h1>
        <div class="pic-wrapper" ref="picWrapper">
          <ul class="pic-list" ref="picList">
            <li class="pic-item" v-for="(pic, index) in seller.pics" :key="index">
              <img :src="pic" width="120" height="90">
            </li>
          </ul>
        </div>
      </div>
      <split></split>
      <div class="info">
        <h1 class="title border-bottom-1px">商家信息</h1>
        <ul>
          <li class="info-item" v-for="(info, index) in seller.infos" :key="index">{{ info }}</li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
  import Star from 'components/star/star'
  import Split from 'components/split/split'
  import BScroll from 'better-scroll'
  import { saveToLocal, loadFromLocal } from '../../common/js/storage'

  export default {
    name: 'seller',
    props: {
      seller: {
        type: Object
      }
    },
    data() {
      return {
        favorite: (() => {
          return loadFromLocal(this.seller.id, 'favorite', false)
        })()
      }
    },
    computed: {
      favoriteText() {
        return this.favorite ? '已收藏' : '收藏'
      }
    },
    created() {
      this.classMap = ['decrease', 'discount', 'invoice', 'special', 'guarantee']
    },
    mounted() {
      this._initScroll()
      this._initPics()
    },
    watch: {
      seller() {
        this._initScroll()
        this._initPics()
      }
    },
    methods: {
      _initScroll() {
        if (!this.scroll) {
          this.scroll = new BScroll(this.$refs.seller, {
            click: true
          })
        } else {
          this.scroll.refresh()
        }
      },
      _initPics() {
        if (this.seller.pics) {
          const picWidth = 120
          const margin = 6
          const width = (picWidth + margin) * this.seller.pics.length - margin
          this.$refs.picList.style.width = width + 'px'
          this.$nextTick(() => {
            if (!this.picScroll) {
              this.picScroll = new BScroll(this.$refs.picWrapper, {
                scrollX: true,
                eventPassthrough: 'vertical'
              })
            } else {
              this.picScroll.refresh()
            }
          })
        }
      },
      toggleFavorite() {
        this.favorite = !this.favorite
        saveToLocal(this.seller.id, 'favorite', this.favorite)
      }
    },
    components: {
      Split,
      Star
    }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin.styl"

  .seller
    position absolute
    top 174px
    bottom 0
    left 0
    width 100%
    overflow hidden
    .overview
      position relative
      padding 18px
      .title
        margin-bottom 8px
        line-height 14px
        color rgb(7, 17, 27)
        font-size 14px
      .desc
        display flex
        align-items center
        padding-bottom 18px
        font-size 0
        .star
          margin-right 8px
        .text
          margin-right 12px
          line-height 18px
          font-size 10px
          color rgb(77, 85, 93)
      .remark
        display flex
        padding-top 18px
        .block
          flex 1
          text-align center
          border-right 1px solid rgba(7, 17, 27, 0.1)
          &:last-child
            border none
          h2
            margin-bottom 4px
            line-height 10px
            font-size 10px
            color rgb(147, 153, 159)
          .content
            line-height 24px
            font-size 10px
            color rgb(7, 17, 27)
            .stress
              font-size 24px
      .favorite
        position absolute
        width 50px
        right 11px
        top 18px
        text-align center
        .icon-favorite
          display: block
          margin-bottom 4px
          line-height 24px
          font-size 24px
          color #d4d6d9
          &.active
            color rgb(240, 20, 20)
        .text
          line-height 10px
          font-size 10px
          color rgb(77, 85, 93)
    .bulletin
      padding 18px 18px 0 18px
      line-height 14px
      color rgb(7,17, 27)
      font-size 14px
      .content-wrapper
        padding 0 12px 16px 12px
        .content
          line-height 24px
          font-size 12px
          color rgb(240, 20, 20)
      .supports
        .support-item
          padding 16px 12px
          font-size 0
          &:last-child
            border-none()
          .icon
            display inline-block
            width 16px
            height 16px
            vertical-align top
            margin-right 6px
            background-size 16px 16px
            background-repeat no-repeat
            &.decrease
              bg-image('decrease_4')
            &.discount
              bg-image('discount_4')
            &.guarantee
              bg-image('guarantee_4')
            &.invoice
              bg-image('invoice_4')
            &.special
              bg-image('special_4')
          .text
            line-height 16px
            font-size 12px
            color rgb(7, 17, 27)
    .pics
      padding 18px
      .title
        margin-bottom 8px
        line-height 14px
        color rgb(7, 17, 27)
        font-size 14px
      .pic-wrapper
        width 100%
        overflow hidden
        white-space nowrap
        .pic-list
          font-size 0
          .pic-item
            display inline-block
            margin-right 6px
            width 120px
            height 90
            &:last-child
              margin 0
    .info
      padding 18px 18px 0 18px
      color rgb(7, 17, 27)
      .title
        padding-bottom 12px
        line-height 14px
        font-size 14px
      .info-item
        padding 16px 12px
        line-height 16px
        font-size 12px
        &:last-child
          border-none()
</style>
