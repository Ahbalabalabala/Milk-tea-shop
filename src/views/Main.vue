<template>
  <div class="main">
    <div class="main-content">
      <!-- 二级路由 -->
      <!-- include="Home,Menu" Home组件和Menu组件被缓存 -->
      <keep-alive include="Home,Menu">
        <router-view></router-view>
      </keep-alive>
    </div>

    <van-tabbar v-model="tabbarIndex" active-color="#0C34BA" :route="true">
      <van-tabbar-item
        v-for="(item, index) in tabbarData"
        :key="index"
        :to="item.url"
      >
        <span>{{ item.title }}</span>
        <template #icon="props">
          <img :src="props.active ? item.icon.active : item.icon.inactive" />
        </template>
      </van-tabbar-item>
    </van-tabbar>
  </div>
</template>

<script>
import BScroll from "better-scroll";
export default {
  data() {
    return {
      tabbarIndex: 0,
      isLogin: false,
      tabbarData: [
        {
          title: "首页",
          icon: {
            active: require("../assets/images/home_active.png"),
            inactive: require("../assets/images/home.png"),
          },
          url: {
            name: "Home",
          },
        },
        {
          title: "菜单",
          icon: {
            active: require("../assets/images/menu_active.png"),
            inactive: require("../assets/images/menu.png"),
          },
          url: {
            name: "Menu",
          },
        },
        {
          title: "购物袋",
          icon: {
            active: require("../assets/images/shopbag_active.png"),
            inactive: require("../assets/images/shopbag.png"),
          },
          url: {
            name: "Shopbag",
          },
        },
        {
          title: "我的",
          icon: {
            active: require("../assets/images/my_active.png"),
            inactive: require("../assets/images/my.png"),
          },
          url: {
            name: "My",
          },
          url1: {
            name: "Login",
          },
        },
      ],
    };
  },
  // mounted() {
  //   this.scrolling();
  // },
  // updataed(){
  //   this.scrolling();
  // },
  methods: {
    //界面滚动
    // scrolling() {
    //   if (!this.Scroll) {
    //     this.Scroll = new BScroll(this.$refs.scrollbar, {
    //       click: true, // 允许BScroll内部元素被点击,默认不允许点击
    //     });
    //   } else {
    //     this.Scroll.refresh();
    //   }
    // },
  },
};
</script>

<style lang="less" scoped>
.main {
  width: 100vw;
  height: 100vh;
  position: relative;
  .van-tabbar {
    bottom: -3px;
  }
  .main-content{
    width: 100%;
    position: absolute;
    top: 0px;
    bottom: 45px;
  }
}
</style>