<template>
  <div class="menu">
    <div class="menu-search">
      <van-search
        @focus="goSearch"
        placeholder="请输入商品名称"
        shape="round"
      />
    </div>
    <div class="menu-box">
      <div class="menu-list clearfix" :style="{ width: width + 'rem' }">
        <div
          :ref="'menu' + index"
          class="menu-list-item fl"
          v-for="(item, index) in typeData"
          :key="index"
          @click="toggleType(item)"
        >
          <div class="menu-content">
            <div class="img-box">
              <img
                class="auto-img"
                :src="item.isActive ? item.activeIcon : item.icon"
                alt
              />
            </div>
            <div class="text" :class="{ active: item.isActive }">
              {{ item.typeDesc }}
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="menu-product" ref="menuproduct">
      <ul>
        <div
          class="product-list"
          :class="{ 'not-first': index > 0 }"
          v-for="(item, index) in products"
          :key="index"
          @click="viewProductInfo(item.pid)"
        >
          <div class="pro-box">
            <div class="pro-img">
              <img class="auto-img" :src="item.smallImg" alt />
            </div>
            <div class="pro-info">
              <div class="pro-text fl">
                <div class="ch-name">{{ item.name }}</div>
                <div class="en-name">{{ item.enname }}</div>
              </div>
              <div class="pro-price fr">￥{{ item.price }}</div>
            </div>
          </div>
        </div>
      </ul>
    </div>
  </div>
</template>

<script>
import "../../assets/less/menu.less";
import BScroll from "better-scroll";
export default {
  //组件名称
  name: "Menu",

  data() {
    return {
      width: 0,
      typeData: [],
      products: [],
    };
  },

  created() {
    //获取商品类型数据
    this.getTypeData();
  },
  updated(){
    this.scrolling()
  },
  methods: {
    // 进入搜索界面
    goSearch() {
      this.$router.push({ name: "Search" });
    },

    //获取商品类型数据
    getTypeData() {
      this.$toast.loading({
        message: "加载中...",
        forbidClick: true,
        duration: 0,
        loadingType: "spinner",
      });
      this.axios({
        method: "GET",
        url: "/type",
        //如果get请求，参数需要放在params, 如果是post请求,参数需要放在data
        params: {
          appkey: this.appkey,
        },
      })
        .then((result) => {
          this.$toast.clear();

          if (result.data.code == 400) {
            let iconData = [
              {
                type: "latte",
                icon: require("../../assets/images/icons_05.gif"),
                activeIcon: require("../../assets/images/icons_19.gif"),
              },
              {
                type: "coffee",
                icon: require("../../assets/images/icons_03.gif"),
                activeIcon: require("../../assets/images/icons_18.gif"),
              },
              {
                type: "rena_ice",
                icon: require("../../assets/images/icons_07.gif"),
                activeIcon: require("../../assets/images/icons_20.gif"),
              },
            ];

            result.data.result.map((v) => {
              for (let i = 0; i < iconData.length; i++) {
                if (v.type == iconData[i].type) {
                  v.icon = iconData[i].icon;
                  v.activeIcon = iconData[i].activeIcon;
                  v.isActive = false;
                  break;
                }
              }
            });

            result.data.result.unshift({
              type: "isHot",
              typeDesc: "推荐",
              icon: require("../../assets/images/icons_09.gif"),
              activeIcon: require("../../assets/images/icons_21.gif"),
              isActive: true,
            });

            this.typeData = result.data.result;

            //根据商品类型获取商品数据
            this.getProductsByType(this.typeData[0]);

            this.$nextTick(() => {
              let width = this.$refs.menu0[0].clientWidth;

              this.width = (this.typeData.length * width) / 37.5;
            });
          }
        })
        .catch((err) => {
          this.$toast.clear();
        });
    },

    //切换商品类型
    toggleType(item) {
      if (item.isActive) {
        return;
      }

      this.isLoading = true;

      for (let i = 0; i < this.typeData.length; i++) {
        if (this.typeData[i].isActive) {
          this.typeData[i].isActive = false;
          break;
        }
      }

      item.isActive = true;

      //根据商品类型获取商品数据
      this.getProductsByType(item);
    },

    //根据商品类型获取商品数据
    getProductsByType(item) {
      this.products = [];
      //

      //如果存在缓存数据,则在缓存获取商品数据,不发起请求
      let data = sessionStorage.getItem(item.type);
      if (data) {
        this.isLoading = false;

        this.products = JSON.parse(data);

        return;
      }

      let params = {
        key: "type",
        value: item.type,
        appkey: this.appkey,
      };

      if (item.type == "isHot") {
        params.key = "isHot";
        params.value = 1;
      }

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
        params,
      })
        .then((result) => {
          this.$toast.clear();

          if (result.data.code == 500) {
            this.isLoading = false;

            this.products = result.data.result;

            //将商品数据缓存
            sessionStorage.setItem(item.type, JSON.stringify(this.products));
          }
        })
        .catch((err) => {
          this.$toast.clear();
        });
    },

    //查看商品详情
    viewProductInfo(pid) {
      this.$router.push({ name: "Detail", query: { pid } });
    },
    //界面滚动
    scrolling() {
      this.$nextTick(() => {
        if (!this.Scroll) {
          this.Scroll = new BScroll(this.$refs.menuproduct, {
            click: true, // 允许BScroll内部元素被点击,默认不允许点击
          });
        } else {
          this.Scroll.refresh();
        }
        console.log("滚动+");
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
