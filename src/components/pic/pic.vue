<template>
  <div class="pic" v-if="picinfo.length">
    <!-- <div class="pic_bg" v-if="globalInfo.contentInfo" :style="{backgroundColor:globalInfo.contentInfo.bgcolor, opacity : globalInfo.contentInfo.opacity}"></div> -->
    <div class="pic_content g-content" @scroll="getNewData">
      <picList @showswiper= "showSwiper" v-for="(item, index) in picinfo" :key="item.id" :data-index="index" :data-info="item"></picList>
    </div>
    <transition name="fade-scale">
      <div class="image_detail" v-show="showImageDetail">
        <swiper :options="swiperOption" ref="mySwiper" v-if="picListInfo.detail && globalInfo.contentInfo">  
                <!-- 这部分放你要渲染的那些内容 -->  
                <swiper-slide v-for="(item, index) in picListInfo.detail" :key="item.id">
                  <!-- <div class="swiper-lazy-preloader"></div> -->
                  <img class="image_info" :id="'detail_' + index" :src="item.url" alt="未曾遗忘的青春" :data-disc="picListInfo.disc" :data-title="picListInfo.place">
                </swiper-slide>
                <!-- 这是轮播的小圆点 -->  
                <div class="swiper-pagination" slot="pagination"></div>  
            </swiper>
            <div class="colse_image_detail" @click.stop="hideSwiper">
              <div class="bg_colse_image_detail">
                <i class="icon-close"></i>
              </div>
            </div>
            <div class="set_Bg" @click="setBg">
              设置为壁纸
            </div>
            <div class="bg_info" :style="{backgroundColor:globalInfo.contentInfo.bgcolor,opacity:globalInfo.contentInfo.opacity}"></div>
      </div>
    </transition>
  </div>
</template>
<script src="http://www.daiwei.org/global/js/jquery.mCustomScrollbar.min.js"></script>
<script>
  // import Vue from 'vue'
  import store from './../../store'
  import line from './../common/line/line.vue'
  import picList from './picList/picList.vue'
  import fecth from './../../utils/fecth.js'
  import { swiper, swiperSlide } from 'vue-awesome-swiper'
  // import $ from 'jquery'
  // import 'jquery-mousewheel'
  // import 'malihu-custom-scrollbar-plugin'
  // require('jquery-mousewheel')
  // require('malihu-custom-scrollbar-plugin')
  // require('malihu-custom-scrollbar-plugin')($)

  export default {
    data () {
      return {
        picinfo: [],
        // 获取数据的索引
        getPicDataIndex: 0,
        showImageDetail: false,
        overflowType: 'initial',
        imageBg: '',
        currentPicLists: {},
        swiperOption: {
          notNextTick: false,
          // 是一个组件自有属性，如果notNextTick设置为true，组件则不会通过NextTick来实例化swiper，也就意味着你可以在第一时间获取到swiper对象，假如你需要刚加载遍使用获取swiper对象来做什么事，那么这个属性一定要是true
          autoplay: 30000,
          grabCursor: true,
          setWrapperSize: true,
          autoHeight: true,
          pagination: '.swiper-pagination',
          paginationClickable: true,
          mousewheelControl: true,
          autoplayDisableOnInteraction: false,
          observeParents: true,
          observer: true, // 修改swiper自己或子元素时，自动初始化swiper
          loop: false,
          initialSlide: 0,
          // lazyLoading: true,
          // lazyLoadingOnTransitionStart: true,
          onSlideChangeEnd: swiper => {
            // 这个位置放swiper的回调方法
            this.page = swiper.realIndex + 1
            this.index = swiper.realIndex
          },
          onClick: swiper => {
            // const index = swiper.activeIndex
            // alert(swiper.slides[index].getAttribute('songlist'))
          }
        }
      }
    },
    computed: {
      globalInfo () {
        return store.getters.getGlobalInfo
      },
      swiper () {
        return this.$refs.mySwiper.swiper
      },
      picListInfo () {
        return store.getters.getPicList
      },
      getFixedImageInfo () {
        return store.getters.getGlobalInfo.showBingImage
      }
    },
    methods: {
      fetchData () {
        const getImageConditions = 'http://www.daiwei.org/vue/server/home.php?inAjax=1&do=getImageCondition'
        fecth.post(getImageConditions, {index: this.getPicDataIndex, count: 5}).then((res) => {
          res.data.forEach((value, index, array) => {
            this.picinfo.push(value)
          })
          if (res.data === 0) {
            this.getPicDataIndex ++
          }
        }, (err) => {
          alert(err)
        })
      },
      getNewData () {
        const pContent = document.querySelector('.pic_content')
        // 滚动的高加可视化的高 的 和   应该等于 元素的scrollHeight  才加载数据
        const pContentSH = pContent.scrollTop + pContent.offsetHeight
        if (pContent.scrollHeight === pContentSH) {
          this.getPicDataIndex = this.getPicDataIndex + 1
          this.fetchData()
        } else {
          return
        }
      },
      showSwiper (index) {
        this.showImageDetail = true
        try {
          // alert(this.swiper)
          this.swiper.activeIndex = index
        } catch (e) {
          // alert(1)
        }
        // this.$nextTick(() => {
        //   this.swiper.activeIndex = index
        // })
      },
      setBg () {
        if (this.getFixedImageInfo) {
          alert('需要在设置页面将背景图设置自定义才可以使用状态图片作为背景哦！')
          return
        }
        const el = document.getElementById('detail_' + this.swiper.activeIndex)
        const data = {
          url: el.getAttribute('src'),
          disc: el.getAttribute('data-disc'),
          title: el.getAttribute('data-title')
        }
        store.dispatch({
          type: 'set_FixedImageInfo',
          data: data
        })
        localStorage.setItem('fixedImageBg', JSON.stringify(store.getters.getFixedImageInfo))
      },
      hideSwiper () {
        this.showImageDetail = false
      }
    },
    watch: {
      showImageDetail (val, oldVal) {
        if (val === true) {
          this.overflowType = `hidden`
        } else {
          this.overflowType = `initial`
        }
      }
      // swiper (newval, oldval) {
      //   alert(newval)
      // }
      // picListInfo (val, oldval) {
      //   alert(JSON.stringify(val))
      // }
    },
    components: {
      'v-line': line,
      picList,
      swiper,
      swiperSlide
    },
    mounted () {
      this.fetchData()
      // setTimeout(() => {
      //   $('.pic_content').mCustomScrollbar({
      //     theme: 'dark'
      //   })
      // }, 100)
      // $('.pic_content').mCustomScrollbar({
      //   theme: 'dark'
      // })
      // this.$nextTick(() => { $('.pic_content').mCustomScrollbar() })
      // alert(this.$refs.abc)
      // alert(store.getters.getGlobalInfo)
      // this.swiper.slideTo(3, 1000, false)
    }
  }
</script>
<style lang="stylus" rel="stylesheet/stylus">
  @import '../../common/stylus/global.styl'
  @import '../../common/stylus/style.styl'             
  .pic
    position:fixed
    top:0
    left:0;
    bottom:0
    width:100%
    right:0
    border-radius:12px
    .pic_bg
      position:absolute
      top:0
      left:0;
      bottom:0
      width:100%
      right:0
      z-index:-1
    .pic_content
      position:fixed
      // top:100px
      // bottom:50px
      // left:50%
      max-width:1240px
      width:100%
      transform:translate3d(-50%,0,0)
      margin:0 auto
      overflow-y:auto
      box-sizing:border-box
      -webkit-overflow-scrolling: touch
      padding:15px;
      box-sizing:border-box
    .image_detail
      position:fixed
      top:100px
      bottom:50px
      left:50%
      max-width:1240px
      width:100%
      transform:translate3d(-50%,0,0) scale(1)
      transform-origin:center center
      &.fade-scale-enter-active,&.fade-scale-leave-active
        transition: all 0.5s 
      &.fade-scale-enter,&.fade-scale-leave-to
        opacity:0
        transform:translate3d(-50%,0,0) scale(0)
      .swiper-container
        height:100%
        overflow:hidden
        .swiper-wrapper
          height:100%
          display:flex
          align-items:center
          .swiper-slide
            width:100%
            height: 100%
            overflow:hidden
            display:flex
            align-items:center
            img
              width:auto
              height:auto
              max-width:100%
              max-height:100%
              background:#000
        .swiper-pagination
        .swiper-pagination-bullet
          margin:0 4px
          &.swiper-pagination-bullet-active
            background:$active_color!important
      .bg_info
        position:absolute
        top:0
        left:0
        bottom:0
        right:0
        z-index:-1
      .colse_image_detail
        position:absolute
        width:80px
        height:80px
        line-height:80px
        text-align:center
        top:0
        right:0
        z-index:2
        cursor:pointer
        i
          font-size:46px
          color:$text_color
          vertical-align: middle
        .bg_colse_image_detail
          width:100%
          height:100%
          z-index:-1
      .set_Bg
        color:$text_color
        position:absolute
        z-index:1
        bottom:10px
        right:10px
        font-size:12px
        padding:5px 10px
        background:rgba(0,0,0,0.1)
        cursor:pointer
        &:hover
          background:rgba(0,0,0,0.4)
      .bg_image_info
        display:block
        width:100%
        height:100%
      .image_info
        height:100%
        display:block
        margin:0 auto
</style>
