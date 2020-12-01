<template>
  <div class="search">
    <div class="search-box">
      <van-nav-bar @click-left="goback" @click-right="searchClear" right-text="取消" left-arrow left-text="返回">
        <template #title>
          <van-search @input="searchProductData" placeholder="请输入商品名称" shape="round" />
        </template>
      </van-nav-bar>
    </div>

    <div class="menu-product">
        <div
          class="product-list"
          v-for="(item,index) in searchData"
          :key="index"
          :class="{'not-first': index > 0}"
          @click="productDetail(item.pid)"
        >
          <div class="pro-box">
            <div class="pro-img">
              <img class="auto-img" :src="item.smallImg" alt />
            </div>
            <div class="pro-info">
              <div class="pro-text fl">
                <div class="ch-name">{{item.name}}</div>
                <div class="en-name">{{item.enname}}</div>
              </div>
              <div class="pro-price fr">￥{{item.price}}</div>
            </div>
          </div>
        </div>
    </div>
  </div>
</template>

<script>
import "../assets/less/search.less";
export default {
  data() {
    return {
      //是否显示骨架屏
      isLoading: true,
      searchData: [],
    };
  },

  created() {
    //   this.searchProductData();
  },

  methods: {
    goback() {
      this.$router.go(-1);
    },
    // 搜索商品
    searchProductData(value) {
      this.$toast.loading({
        message: "加载中...",
        forbidClick: true,
        duration: 0,
        loadingType: "spinner",
      });
      this.axios({
        method: "GET",
        url: "/search",
        //如果get请求，参数需要放在params, 如果是post请求,参数需要放在data
        params: {
          appkey: this.appkey,
          name: value,
        },
      })
        .then((result) => {
          this.$toast.clear();
          
          if ((result.data.code = "Q001")) {
            this.searchData = result.data.result;
          }
        })
        .catch((err) => {
          this.$toast.clear();
          
        });
    },

    // 清空列表
    searchClear(){
        this.searchData = [];
    },

    // 商品详情
    productDetail(pid){
        
        this.$router.push({name: "Detail", query: {pid}})
    }
  },
};
</script>
