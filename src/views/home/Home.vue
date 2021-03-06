<template>
  <div id="home">
    <nav-bar class="home-nav"
      ><template v-slot:center>
        <div>
          购物街
        </div>
      </template></nav-bar
    >
    <tab-control
      :titles="['流行', '新款', '精选']"
      @tabClick="tabClick"
      ref="tabControlOuter"
      class="tab-control-fixed"
      v-show="isFixedTabBar"
    />
    <scroll
      class="content"
      ref="scroll"
      :probe-type="3"
      :pull-up-load="true"
      @scroll="contentScroll"
      @pullingUp="loadMore"
    >
      <home-swiper
        :banners="banners"
        @swiperImageLoad="swiperImageLoad"
      ></home-swiper>
      <home-recommends :recommends="recommends"></home-recommends>
      <home-feature-view></home-feature-view>
      <tab-control
        :titles="['流行', '新款', '精选']"
        @tabClick="tabClick"
        ref="tabControlInner"
        class="tab-control-inner"
      ></tab-control>
      <goods-list :goods="showGoods" class="goods-list"></goods-list>
    </scroll>

    <back-top @click.native="backTopClick" v-show="isShowBackTop"></back-top>
  </div>
</template>

<script>
import HomeSwiper from "./childHome/HomeSwiper";
import HomeRecommends from "./childHome/HomeRecommends";
import HomeFeatureView from "./childHome/HomeFeatureView";

import NavBar from "components/common/navbar/NavBar";
import TabControl from "components/content/tabControl/TabControl";
import GoodsList from "components/content/goods/GoodsList";
import Scroll from "components/common/scroll/Scroll";

import { getHomeMultidata, getHomeGoods } from "network/home";
// import { debounce } from "common/utils";
import { itemListenerMixin, backTopMixin } from "common/mixin";

export default {
  name: "Home",
  data() {
    return {
      banners: [],
      recommends: [],
      goods: {
        pop: { page: 1, list: [] },
        new: { page: 1, list: [] },
        sell: { page: 1, list: [] },
      },
      currentTab: "pop",
      tabOffsetTop: 0,
      isFixedTabBar: false,
      scrollY: 0,
    };
  },
  computed: {
    showGoods() {
      return this.goods[this.currentTab].list;
    },
  },
  components: {
    NavBar,
    HomeSwiper,
    HomeRecommends,
    HomeFeatureView,
    TabControl,
    GoodsList,
    Scroll,
  },
  mixins: [itemListenerMixin, backTopMixin],
  created() {
    // 请求多个数据
    this.getHomeMultidata();

    // 请求商品数据
    this.getHomeGoods("pop");
    this.getHomeGoods("new");
    this.getHomeGoods("sell");
  },
  activated() {
    this.$refs.scroll.scrollTo(0, this.scrollY);
    this.$refs.scroll.refresh();
  },
  deactivated() {
    // 保存Y值
    this.scrollY = this.$refs.scroll.getScrollY();
  },
  methods: {
    /**
     * 事件监听
     */
    tabClick(index) {
      switch (index) {
        case 0:
          this.currentTab = "pop";
          break;
        case 1:
          this.currentTab = "new";
          break;
        case 2:
          this.currentTab = "sell";
      }
      // 同意两个栏的当前高亮
      this.$refs.tabControlInner.currentIndex = this.$refs.tabControlOuter.currentIndex = index;
      // 当点击tabcontrl组件时，回到商品栏顶部
      this.$refs.scroll.scrollTo(0, -this.tabOffsetTop);
    },
    // 监听滚动
    contentScroll(position) {
      // 通过混入的函数，判断是否显示回顶图标
      this.listenShowBackTop(position);
      this.isFixedTabBar = -position.y > this.tabOffsetTop;
    },
    // 上拉加载更多
    loadMore() {
      this.getHomeGoods(this.currentTab);
    },
    swiperImageLoad() {
      this.tabOffsetTop = this.$refs.tabControlInner.$el.offsetTop;
    },

    /**
     * 网络请求
     */
    getHomeMultidata() {
      getHomeMultidata().then((res) => {
        [this.banners, this.recommends] = [
          res.data.banner.list,
          res.data.recommend.list,
        ];
      });
    },
    getHomeGoods(type) {
      const page = this.goods[type].page;
      getHomeGoods(type, page).then((res) => {
        this.goods[type].list.push(...res.data.list);
        this.goods[type].page++;
      });
    },
  },
};
</script>

<style scoped>
#home {
  position: relative;
  height: 100vh;
}

.home-nav {
  background-color: var(--color-tint);
  color: white;
  font-weight: 700;
}

.goods-list {
  z-index: -1;
}

.content {
  position: absolute;
  top: 44px;
  bottom: 49px;
  left: 0;
  right: 0;

  overflow: hidden;
}
.tab-control-inner {
  background-color: white;
}

.tab-control-fixed {
  background-color: white;
  position: relative;
  z-index: 9;
}
</style>
