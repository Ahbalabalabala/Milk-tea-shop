<template>
  <div class="shopbag">
    <van-nav-bar
      title="购物袋"
      left-text="返回"
      left-arrow
      :fixed="true"
      @click-left="back"
      @click-right="editProduct"
    >
      <template #right v-if="shopbagData.length > 0">
        <div>{{ isEdit ? "完成" : "编辑" }}</div>
      </template>
    </van-nav-bar>
    <div v-if="shopbagData.length > 0">
      <div class="shopbag-content" ref="shopbagcontent">
        <ul>
          <div class="shopbag-bg"></div>
          <van-swipe-cell
            :disabled="isEdit"
            v-for="(item, index) in shopbagData"
            :key="index"
          >
            <van-cell :border="false">
              <div class="clearfix cell-box">
                <div class="fl check">
                  <van-checkbox
                    v-model="item.isCheck"
                    icon-size="24px"
                    checked-color="#0C34BA"
                    @change="simpleSelect"
                  ></van-checkbox>
                </div>
                <div class="fl pro-img" @click="viewProductInfo(item.pid)">
                  <img class="auto-img" :src="item.small_img" alt />
                </div>
                <div class="fl pro-info">
                  <div class="text-box">
                    <div class="clearfix">
                      <div class="fl text-name">{{ item.name }}</div>
                      <div class="fl text-rule">{{ item.rule }}</div>
                    </div>
                    <div class="text-enname">{{ item.enname }}</div>
                  </div>
                  <div class="price-box">
                    <div class="fl price">￥{{ item.price }}</div>
                    <div class="fr">
                      <van-stepper
                        v-model="item.count"
                        theme="round"
                        button-size="24"
                        disable-input
                        @change="modifyCount(item)"
                      />
                    </div>
                  </div>
                </div>
              </div>
            </van-cell>

            <template #right>
              <van-button
                square
                color="#0C34BA"
                text="删除"
                @click="removeOneShopbag(item.sid)"
              />
            </template>
          </van-swipe-cell>
        </ul>
      </div>

      <div>
        <van-submit-bar
          v-show="!isEdit"
          :price="total"
          button-text="提交订单"
          button-color="#0C34BA"
          @submit="commit"
        >
          <van-checkbox
            @click="allSelect"
            v-model="allCheck"
            icon-size="24px"
            checked-color="#0C34BA"
            >全选</van-checkbox
          >
        </van-submit-bar>

        <van-submit-bar
          v-show="isEdit"
          button-text="删除选择"
          button-color="#0C34BA"
          @submit="removeMoreShopbag"
        >
          <van-checkbox
            v-model="allCheck"
            @click="allSelect"
            icon-size="24px"
            checked-color="#0C34BA"
            >全选</van-checkbox
          >
        </van-submit-bar>
      </div>
    </div>
    <div v-else>
      <van-empty description="购物袋空空如也,去逛一逛!">
        <van-button round color="#0C34BA" class="bottom-button" @click="goMenu"
          >去逛一逛</van-button
        >
      </van-empty>
    </div>
  </div>
</template>

<script>
import "../../assets/less/shopbag.less";
import { utils } from "../../assets/js/utils";
import BScroll from "better-scroll";
export default {
  data() {
    return {
      // 是否为编辑状态
      isEdit: false,
      // 是否全选
      allCheck: false,
      // 购物袋数据
      shopbagData: [],
      // 总额金额
      total: 0,
    };
  },
  created() {
    //查询购物袋数据
    this.findShopbag();
    
  },
  updated() {
    this.scrolling();
  },
  methods: {
    back() {
      this.$router.go(-1);
    },
    //回到菜单
    goMenu() {
      this.$router.push({ name: "Menu" });
    },

    //查看商品详情
    viewProductInfo(pid) {
      this.$router.push({ name: "Detail", query: { pid } });
    },

    //提交订单
    commit() {
      //查找需要购买的商品, sid
      let sids = [];
      this.shopbagData.map((v) => {
        if (v.isCheck) {
          sids.push(v.sid);
        }
      });

      // 如果商品的sid都没传过去则没有购买的商品
      if (sids.length == 0) {
        this.$toast("请选择购买的商品");
        return;
      }

      // 将数组的值用“-”连接起来
      sids = sids.join("-");

      this.$router.push({ name: "Pay", query: { sids } });

      //
    },

    //编辑商品
    editProduct() {
      this.isEdit = !this.isEdit;
    },

    //查询购物袋数据
    findShopbag() {
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
        url: "/findAllShopcart",
        params: {
          appkey: this.appkey,
          tokenString,
        },
      })
        .then((result) => {
          this.$toast.clear();

          if (result.data.code == 5000) {
            result.data.result.map((v) => {
              v.isCheck = false;
            });

            this.shopbagData = result.data.result;
          } else {
            this.$toast(result.data.msg);
          }
          this.scrolling()
        })
        .catch((err) => {
          this.$toast.clear();
        });
    },

    //全选
    allSelect() {
      this.shopbagData.map((v) => {
        v.isCheck = this.allCheck;
      });

      this.sum();
    },

    //单选
    simpleSelect() {
      this.sum();

      for (let i = 0; i < this.shopbagData.length; i++) {
        if (!this.shopbagData[i].isCheck) {
          this.allCheck = false;
          return;
        }
      }

      this.allCheck = true;
    },

    //修改商品数量
    modifyCount(item) {
      //获取token字符串
      let tokenString = localStorage.getItem("CSTK");
      if (!tokenString) {
        return;
      }

      //参数序列化
      let data = utils.queryString({
        appkey: this.appkey,
        tokenString,
        sid: item.sid,
        count: item.count,
      });

      this.$toast.loading({
        message: "加载中...",
        forbidClick: true,
        duration: 0,
        loadingType: "spinner",
      });

      this.axios({
        method: "POST",
        url: "/modifyShopcartCount",
        data,
      })
        .then((result) => {
          this.$toast.clear();

          this.sum();
          if (result.data.code !== 6000) {
            this.$toast(result.data.msg);
          }
        })
        .catch((err) => {
          this.$toast.clear();
        });
    },

    //删除购物袋数据
    removeShopbag(sids) {
      //sids: sid的集合，类型：array

      //获取token字符串
      let tokenString = localStorage.getItem("CSTK");
      if (!tokenString) {
        return;
      }

      //参数序列化
      let data = utils.queryString({
        appkey: this.appkey,
        tokenString,
        sids: JSON.stringify(sids),
      });

      this.$toast.loading({
        message: "加载中...",
        forbidClick: true,
        duration: 0,
        loadingType: "spinner",
      });

      return this.axios({
        method: "POST",
        url: "/deleteShopcart",
        data,
      });
    },

    //单个删除购物袋
    removeOneShopbag(sid, index) {
      this.removeShopbag([sid])
        .then((result) => {
          this.$toast.clear();

          if (result.data.code == 7000) {
            this.shopbagData.splice(index, 1);
          }

          this.$toast(result.data.msg);

          this.sum();
        })
        .catch((err) => {
          this.$toast.clear();
        });
    },

    //删除多个购物袋
    removeMoreShopbag() {
      //查找选择购物袋
      let sids = [];
      this.shopbagData.map((v) => {
        if (v.isCheck) {
          sids.push(v.sid);
        }
      });

      this.removeShopbag(sids)
        .then((result) => {
          this.$toast.clear();

          if (result.data.code == 7000) {
            for (let i = this.shopbagData.length - 1; i >= 0; i--) {
              if (this.shopbagData[i].isCheck) {
                this.shopbagData.splice(i, 1);
              }
            }

            this.sum();
          }

          this.$toast(result.data.msg);
        })
        .catch((err) => {
          this.$toast.clear();
        });
    },

    //计算总金额
    sum() {
      this.total = 0;
      this.shopbagData.map((v) => {
        if (v.isCheck) {
          this.total += v.count * v.price;
        }
      });

      this.total *= 100;
    },
    //界面滚动
    scrolling() {
      this.$nextTick(() => {
        if(!this.shopbagData.length){
          return
        }
        if (!this.Scroll) {
          this.Scroll = new BScroll(this.$refs.shopbagcontent, {
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