<template>
  <div class="home">
    <van-nav-bar :fixed="true">
      <template #left>
        <div class="home-title">
          {{ timeText }}<span v-if="isLogin">, {{ userInfo.nickName }}</span>
        </div>
      </template>
      <template #right>
        <van-search
          @focus="goSearch"
          placeholder="请输入商品名称"
          shape="round"
        />
      </template>
    </van-nav-bar>
    <div class="scroll-bar" ref="scrollbar">
      <ul>
        <div class="banner-box">
          <van-swipe :autoplay="3000" :show-indicators="false">
            <van-swipe-item v-for="(item, index) in bannerData" :key="index">
              <img class="auto-img" :src="item.bannerImg" alt="" />
              <div
                class="pro-info-banner one-text"
                @click="viewProductInfo(item.pid)"
              >
                {{ item.name }}
              </div>
            </van-swipe-item>
          </van-swipe>
        </div>
        <div class="hot-product">
          <div class="hot-product-title">热门推荐</div>
          <div class="product-box clearfix">
            <div
              class="product-item fl"
              :class="[index % 2 == 0 ? 'p-left' : 'p-right']"
              v-for="(item, index) in hotProducts"
              :key="index"
            >
              <div class="p-img" @click="viewProductInfo(item.pid)">
                <img class="auto-img" :src="item.smallImg" alt="" />
              </div>
              <div class="pro-name one-text">{{ item.name }}</div>
              <div class="clearfix">
                <!-- <div class="en-pro-name one-text fl">heitangbobo</div> -->
                <div class="pro-price fl">￥{{ item.price }}</div>
              </div>
            </div>
          </div>
        </div>
      </ul>
    </div>
  </div>
</template>

<script>
import "../../assets/less/home.less";
import { Search } from "vant";
import BScroll from "better-scroll";

export default {
  name: "Home",
  data() {
    return {
      timeText: "",
      bannerData: [],
      hotProducts: [],
      isLogin: false,
      userInfo: {},
      // Scroll: 0,
    };
  },

  created() {
    this.getTime();
    this.getbannerData();
    this.getHotProducts();
    this.getUserInfo();
    // this.getGoodsFromServer();
    // this.scrolling();
  },
  mounted() {},
  updated() {
    this.scrolling();
  },
  methods: {
    getGoodsFromServer() {
      var p1 = new Promise(() => {
        this.getbannerData();
        // this.scrolling();
      });
      var p2 = new Promise(() => {
        this.getHotProducts();
        // this.scrolling();
      });
      Promise.all([p1, p2]).then((res) => {
        console.log(res, "1");
        // console.log(this.getbannerData(),this.getHotProducts(),this.scrolling());
        this.scrolling();
      });
      // let p1 = new Promise(() => {
      //   this.getbannerData();
      //   this.getHotProducts();
      // })
      //   .then((res) => {})
      //   .then((res) => {
      //     // this.scrolling();
      //   });
    },
    //获取时间段
    getTime() {
      let hours = new Date().getHours();
      if (hours >= 6 && hours < 12) {
        this.timeText = "早上好";
      } else if (hours >= 12 && hours < 18) {
        this.timeText = "下午好";
      } else if (hours >= 18 && hours < 24) {
        this.timeText = "晚上好";
      } else {
        this.timeText = "深夜好";
      }
    },

    // 进入搜索界面
    goSearch() {
      this.$router.push({ name: "Search" });
    },

    //获取banner
    getbannerData() {
      this.$toast.loading({
        message: "加载中...",
        forbidClick: true,
        duration: 0,
        loadingType: "spinner",
      });
      this.axios({
        method: "GET",
        url: "/banner",
        //如果get请求，参数需要放在params, 如果是post请求,参数需要放在data
        params: {
          appkey: this.appkey,
        },
      })
        .then((result) => {
          // console.log("轮播图");
          // this.scrolling();
          this.$toast.clear();

          if (result.data.code == 300) {
            this.bannerData = result.data.result;
          }
          // this.scrolling();
          // this.$forceUpdate();
        })
        .catch((err) => {
          this.$toast.clear();
        });
    },

    //获取推荐商品数据
    getHotProducts() {
      this.$toast.loading({
        message: "加载中...",
        forbidClick: true,
        duration: 0,
        loadingType: "spinner",
      });
      this.axios({
        method: "GET",
        url: "/typeProducts",
        //如果get请求，参数需要放在params, 如果是post请求,参数需要放在data
        params: {
          appkey: this.appkey,
          key: "isHot",
          value: 1,
        },
      })
        .then((result) => {
          this.$toast.clear();

          if (result.data.code == 500) {
            // console.log("商品");
            this.hotProducts = result.data.result;
          }
          // this.scrolling();
          // this.$forceUpdate();
        })
        .catch((err) => {
          this.$toast.clear();
        });
    },

    //查看商品详情
    viewProductInfo(pid) {
      this.$router.push({ name: "Detail", query: { pid } });
    },

    //获取用户信息
    getUserInfo() {
      //获取token字符串
      let tokenString = localStorage.getItem("CSTK");
      if (!tokenString) {
        return;
      }

      this.$toast.loading({
        message: "加载中...",
        forbidClick: true,
        duration: 0,
        loadingType: "spinner",
      });

      this.axios({
        method: "GET",
        url: "/findMy",
        params: {
          appkey: this.appkey,
          tokenString,
        },
      })
        .then((result) => {
          this.$toast.clear();

          if (result.data.code == "A001") {
            if (result.data.result.length > 0) {
              this.isLogin = true;
              this.userInfo = result.data.result[0];
            }
          }
        })
        .catch((err) => {
          this.$toast.clear();
        });
    },
    //界面滚动
    scrolling() {
      this.$nextTick(() => {
        if (!this.Scroll) {
          this.Scroll = new BScroll(this.$refs.scrollbar, {
            click: true, // 允许BScroll内部元素被点击,默认不允许点击
          });
        } else {
          this.Scroll.refresh();
        }
        // console.log("滚动+");
      });
    },
  },
  beforeDestory() {
    // BScroll不属于vue,他是一个有副作用的DOM操作,所以当前组件被卸载时请回收该实例
    if (this.Scroll) {
      this.Scroll.destroy();
    }
  },
};
</script>